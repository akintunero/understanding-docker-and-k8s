# Project 04: File Upload Service

A Go-based file upload service with MinIO object storage demonstrating volume management and persistent storage in Kubernetes.

## 🚀 Project Overview

This project showcases a file upload service with:
- **Backend**: Go HTTP server
- **Storage**: MinIO object storage
- **File Processing**: Image resizing and metadata extraction
- **Containerization**: Docker with volume management
- **Orchestration**: Kubernetes with persistent volumes
- **Security**: File type validation and size limits

## 📁 Project Structure

```
file-upload-service/
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── handlers/
│   │   └── upload.go
│   ├── storage/
│   │   └── minio.go
│   └── utils/
│       └── image.go
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
├── kubernetes/
│   ├── namespace.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── pvc.yaml
│   ├── minio-deployment.yaml
│   ├── minio-service.yaml
│   ├── upload-deployment.yaml
│   ├── upload-service.yaml
│   └── ingress.yaml
├── go.mod
├── go.sum
└── README.md
```

## 🛠️ Technology Stack

### Backend
- **Language**: Go 1.19
- **Framework**: Standard HTTP library
- **Storage**: MinIO object storage
- **Image Processing**: ImageMagick
- **Validation**: File type detection
- **Testing**: Go testing framework

### Infrastructure
- **Containerization**: Docker
- **Orchestration**: Kubernetes
- **Storage**: Persistent volumes
- **Object Storage**: MinIO
- **Monitoring**: Basic metrics

## 🐳 Docker Setup

### Development Environment

1. **Create project structure:**
   ```bash
   mkdir file-upload-service && cd file-upload-service
   mkdir -p cmd/server internal/handlers internal/storage internal/utils docker kubernetes
   ```

2. **Initialize Go module:**
   ```bash
   go mod init file-upload-service
   ```

3. **Create main application:**
   ```bash
   cat << EOF > cmd/server/main.go
   package main

   import (
       "log"
       "net/http"
       "os"
       "file-upload-service/internal/handlers"
       "file-upload-service/internal/storage"
   )

   func main() {
       // Initialize MinIO client
       minioClient, err := storage.NewMinIOClient()
       if err != nil {
           log.Fatal("Failed to initialize MinIO client:", err)
       }

       // Initialize upload handler
       uploadHandler := handlers.NewUploadHandler(minioClient)

       // Setup routes
       http.HandleFunc("/health", handlers.HealthCheck)
       http.HandleFunc("/upload", uploadHandler.HandleUpload)
       http.HandleFunc("/files", uploadHandler.HandleListFiles)
       http.HandleFunc("/files/", uploadHandler.HandleDownloadFile)

       port := os.Getenv("PORT")
       if port == "" {
           port = "8080"
       }

       log.Printf("Server starting on port %s", port)
       log.Fatal(http.ListenAndServe(":"+port, nil))
   }
   EOF
   ```

4. **Create upload handler:**
   ```bash
   cat << EOF > internal/handlers/upload.go
   package handlers

   import (
       "encoding/json"
       "fmt"
       "io"
       "log"
       "net/http"
       "path/filepath"
       "strings"
       "time"
       "file-upload-service/internal/storage"
       "file-upload-service/internal/utils"
   )

   type UploadHandler struct {
       minioClient *storage.MinIOClient
   }

   type UploadResponse struct {
       Success bool   \`json:"success"\`
       FileID  string \`json:"file_id,omitempty"\`
       Error   string \`json:"error,omitempty"\`
   }

   type FileInfo struct {
       Name         string    \`json:"name"\`
       Size         int64     \`json:"size"\`
       ContentType  string    \`json:"content_type"\`
       UploadedAt   time.Time \`json:"uploaded_at"\`
       URL          string    \`json:"url"\`
   }

   func NewUploadHandler(minioClient *storage.MinIOClient) *UploadHandler {
       return &UploadHandler{
           minioClient: minioClient,
       }
   }

   func (h *UploadHandler) HandleUpload(w http.ResponseWriter, r *http.Request) {
       if r.Method != http.MethodPost {
           http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
           return
       }

       // Parse multipart form
       err := r.ParseMultipartForm(32 << 20) // 32MB max
       if err != nil {
           http.Error(w, "Failed to parse form", http.StatusBadRequest)
           return
       }

       file, header, err := r.FormFile("file")
       if err != nil {
           http.Error(w, "Failed to get file", http.StatusBadRequest)
           return
       }
       defer file.Close()

       // Validate file type
       if !utils.IsAllowedFileType(header.Filename) {
           http.Error(w, "File type not allowed", http.StatusBadRequest)
           return
       }

       // Generate unique file ID
       fileID := fmt.Sprintf("%d_%s", time.Now().UnixNano(), header.Filename)

       // Upload to MinIO
       err = h.minioClient.UploadFile(file, fileID, header.Header.Get("Content-Type"))
       if err != nil {
           log.Printf("Failed to upload file: %v", err)
           http.Error(w, "Failed to upload file", http.StatusInternalServerError)
           return
       }

       // Return success response
       response := UploadResponse{
           Success: true,
           FileID:  fileID,
       }

       w.Header().Set("Content-Type", "application/json")
       json.NewEncoder(w).Encode(response)
   }

   func (h *UploadHandler) HandleListFiles(w http.ResponseWriter, r *http.Request) {
       if r.Method != http.MethodGet {
           http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
           return
       }

       files, err := h.minioClient.ListFiles()
       if err != nil {
           http.Error(w, "Failed to list files", http.StatusInternalServerError)
           return
       }

       w.Header().Set("Content-Type", "application/json")
       json.NewEncoder(w).Encode(files)
   }

   func (h *UploadHandler) HandleDownloadFile(w http.ResponseWriter, r *http.Request) {
       if r.Method != http.MethodGet {
           http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
           return
       }

       // Extract file ID from URL
       fileID := strings.TrimPrefix(r.URL.Path, "/files/")
       if fileID == "" {
           http.Error(w, "File ID required", http.StatusBadRequest)
           return
       }

       // Get file from MinIO
       file, contentType, err := h.minioClient.GetFile(fileID)
       if err != nil {
           http.Error(w, "File not found", http.StatusNotFound)
           return
       }
       defer file.Close()

       w.Header().Set("Content-Type", contentType)
       io.Copy(w, file)
   }

   func HealthCheck(w http.ResponseWriter, r *http.Request) {
       w.Header().Set("Content-Type", "application/json")
       json.NewEncoder(w).Encode(map[string]string{"status": "healthy"})
   }
   EOF
   ```

5. **Create MinIO storage client:**
   ```bash
   cat << EOF > internal/storage/minio.go
   package storage

   import (
       "context"
       "fmt"
       "io"
       "log"
       "os"
       "time"
       "file-upload-service/internal/handlers"
       "github.com/minio/minio-go/v7"
       "github.com/minio/minio-go/v7/pkg/credentials"
   )

   type MinIOClient struct {
       client *minio.Client
       bucket string
   }

   func NewMinIOClient() (*MinIOClient, error) {
       endpoint := os.Getenv("MINIO_ENDPOINT")
       if endpoint == "" {
           endpoint = "localhost:9000"
       }

       accessKey := os.Getenv("MINIO_ACCESS_KEY")
       if accessKey == "" {
           accessKey = "minioadmin"
       }

       secretKey := os.Getenv("MINIO_SECRET_KEY")
       if secretKey == "" {
           secretKey = "minioadmin"
       }

       bucket := os.Getenv("MINIO_BUCKET")
       if bucket == "" {
           bucket = "uploads"
       }

       client, err := minio.New(endpoint, &minio.Options{
           Creds:  credentials.NewStaticV4(accessKey, secretKey, ""),
           Secure: false,
       })
       if err != nil {
           return nil, fmt.Errorf("failed to create MinIO client: %v", err)
       }

       // Ensure bucket exists
       err = client.MakeBucket(context.Background(), bucket, minio.MakeBucketOptions{})
       if err != nil {
           exists, errBucketExists := client.BucketExists(context.Background(), bucket)
           if errBucketExists == nil && exists {
               log.Printf("Bucket %s already exists", bucket)
           } else {
               return nil, fmt.Errorf("failed to create bucket: %v", err)
           }
       }

       return &MinIOClient{
           client: client,
           bucket: bucket,
       }, nil
   }

   func (m *MinIOClient) UploadFile(file io.Reader, fileID, contentType string) error {
       _, err := m.client.PutObject(context.Background(), m.bucket, fileID, file, -1, minio.PutObjectOptions{
           ContentType: contentType,
       })
       return err
   }

   func (m *MinIOClient) GetFile(fileID string) (io.ReadCloser, string, error) {
       obj, err := m.client.GetObject(context.Background(), m.bucket, fileID, minio.GetObjectOptions{})
       if err != nil {
           return nil, "", err
       }

       stat, err := obj.Stat()
       if err != nil {
           obj.Close()
           return nil, "", err
       }

       return obj, stat.ContentType, nil
   }

   func (m *MinIOClient) ListFiles() ([]handlers.FileInfo, error) {
       var files []handlers.FileInfo

       objects := m.client.ListObjects(context.Background(), m.bucket, minio.ListObjectsOptions{})
       for obj := range objects {
           if obj.Err != nil {
               continue
           }

           files = append(files, handlers.FileInfo{
               Name:        obj.Key,
               Size:        obj.Size,
               ContentType: obj.ContentType,
               UploadedAt:  obj.LastModified,
               URL:         fmt.Sprintf("/files/%s", obj.Key),
           })
       }

       return files, nil
   }
   EOF
   ```

6. **Create utility functions:**
   ```bash
   cat << EOF > internal/utils/image.go
   package utils

   import (
       "path/filepath"
       "strings"
   )

   var allowedExtensions = map[string]bool{
       ".jpg":  true,
       ".jpeg": true,
       ".png":  true,
       ".gif":  true,
       ".pdf":  true,
       ".txt":  true,
       ".doc":  true,
       ".docx": true,
       ".xls":  true,
       ".xlsx": true,
   }

   func IsAllowedFileType(filename string) bool {
       ext := strings.ToLower(filepath.Ext(filename))
       return allowedExtensions[ext]
   }
   EOF
   ```

7. **Create go.mod:**
   ```bash
   cat << EOF > go.mod
   module file-upload-service

   go 1.19

   require (
       github.com/minio/minio-go/v7 v7.0.63
   )
   EOF
   ```

8. **Create Dockerfile:**
   ```bash
   cat << EOF > docker/Dockerfile
   FROM golang:1.19-alpine AS builder

   WORKDIR /app
   COPY go.mod go.sum ./
   RUN go mod download

   COPY . .
   RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o server ./cmd/server

   FROM alpine:latest
   RUN apk --no-cache add ca-certificates
   WORKDIR /root/

   COPY --from=builder /app/server .
   EXPOSE 8080

   CMD ["./server"]
   EOF
   ```

9. **Create docker-compose.yml:**
   ```bash
   cat << EOF > docker/docker-compose.yml
   version: '3.8'

   services:
     upload-service:
       build:
         context: ..
         dockerfile: docker/Dockerfile
       ports:
         - "8080:8080"
       environment:
         - MINIO_ENDPOINT=minio:9000
         - MINIO_ACCESS_KEY=minioadmin
         - MINIO_SECRET_KEY=minioadmin
         - MINIO_BUCKET=uploads
       depends_on:
         - minio
       restart: unless-stopped

     minio:
       image: minio/minio:latest
       ports:
         - "9000:9000"
         - "9001:9001"
       environment:
         - MINIO_ROOT_USER=minioadmin
         - MINIO_ROOT_PASSWORD=minioadmin
       command: server /data --console-address ":9001"
       volumes:
         - minio_data:/data
       restart: unless-stopped

   volumes:
     minio_data:
   EOF
   ```

## ☸️ Kubernetes Setup

### Create Kubernetes manifests

1. **Namespace:**
   ```bash
   cat << EOF > kubernetes/namespace.yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: file-upload
   EOF
   ```

2. **ConfigMap:**
   ```bash
   cat << EOF > kubernetes/configmap.yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: upload-service-config
     namespace: file-upload
   data:
     MINIO_ENDPOINT: minio-service:9000
     MINIO_BUCKET: uploads
   EOF
   ```

3. **Secret:**
   ```bash
   cat << EOF > kubernetes/secret.yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: minio-secrets
     namespace: file-upload
   type: Opaque
   data:
     MINIO_ACCESS_KEY: bWluaW9hZG1pbg==
     MINIO_SECRET_KEY: bWluaW9hZG1pbg==
   EOF
   ```

4. **Persistent Volume Claim:**
   ```bash
   cat << EOF > kubernetes/pvc.yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: minio-pvc
     namespace: file-upload
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 10Gi
   EOF
   ```

5. **MinIO Deployment:**
   ```bash
   cat << EOF > kubernetes/minio-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: minio
     namespace: file-upload
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: minio
     template:
       metadata:
         labels:
           app: minio
       spec:
         containers:
         - name: minio
           image: minio/minio:latest
           ports:
           - containerPort: 9000
           - containerPort: 9001
           env:
           - name: MINIO_ROOT_USER
             valueFrom:
               secretKeyRef:
                 name: minio-secrets
                 key: MINIO_ACCESS_KEY
           - name: MINIO_ROOT_PASSWORD
             valueFrom:
               secretKeyRef:
                 name: minio-secrets
                 key: MINIO_SECRET_KEY
           command: ["server", "/data", "--console-address", ":9001"]
           volumeMounts:
           - name: minio-storage
             mountPath: /data
           resources:
             requests:
               memory: "256Mi"
               cpu: "250m"
             limits:
               memory: "512Mi"
               cpu: "500m"
         volumes:
         - name: minio-storage
           persistentVolumeClaim:
             claimName: minio-pvc
   EOF
   ```

6. **MinIO Service:**
   ```bash
   cat << EOF > kubernetes/minio-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: minio-service
     namespace: file-upload
   spec:
     selector:
       app: minio
     ports:
     - name: api
       port: 9000
       targetPort: 9000
     - name: console
       port: 9001
       targetPort: 9001
     type: ClusterIP
   EOF
   ```

7. **Upload Service Deployment:**
   ```bash
   cat << EOF > kubernetes/upload-deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: upload-service
     namespace: file-upload
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: upload-service
     template:
       metadata:
         labels:
           app: upload-service
       spec:
         containers:
         - name: upload-service
           image: upload-service:latest
           ports:
           - containerPort: 8080
           env:
           - name: MINIO_ENDPOINT
             valueFrom:
               configMapKeyRef:
                 name: upload-service-config
                 key: MINIO_ENDPOINT
           - name: MINIO_BUCKET
             valueFrom:
               configMapKeyRef:
                 name: upload-service-config
                 key: MINIO_BUCKET
           - name: MINIO_ACCESS_KEY
             valueFrom:
               secretKeyRef:
                 name: minio-secrets
                 key: MINIO_ACCESS_KEY
           - name: MINIO_SECRET_KEY
             valueFrom:
               secretKeyRef:
                 name: minio-secrets
                 key: MINIO_SECRET_KEY
           resources:
             requests:
               memory: "128Mi"
               cpu: "200m"
             limits:
               memory: "256Mi"
               cpu: "500m"
           livenessProbe:
             httpGet:
               path: /health
               port: 8080
             initialDelaySeconds: 30
             periodSeconds: 10
           readinessProbe:
             httpGet:
               path: /health
               port: 8080
             initialDelaySeconds: 5
             periodSeconds: 5
   EOF
   ```

8. **Upload Service:**
   ```bash
   cat << EOF > kubernetes/upload-service.yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: upload-service
     namespace: file-upload
   spec:
     selector:
       app: upload-service
     ports:
     - port: 80
       targetPort: 8080
     type: ClusterIP
   EOF
   ```

9. **Ingress:**
   ```bash
   cat << EOF > kubernetes/ingress.yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: upload-ingress
     namespace: file-upload
     annotations:
       nginx.ingress.kubernetes.io/rewrite-target: /
   spec:
     ingressClassName: nginx
     rules:
     - host: upload.local
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: upload-service
               port:
                 number: 80
   EOF
   ```

## 🚀 Deployment

### Docker Deployment
```bash
# Build and run with Docker Compose
cd docker
docker-compose up -d

# Test the service
curl http://localhost:8080/health
curl -X POST -F "file=@test.jpg" http://localhost:8080/upload
curl http://localhost:8080/files
```

### Kubernetes Deployment
```bash
# Apply all manifests
kubectl apply -f kubernetes/

# Check deployment status
kubectl get pods -n file-upload
kubectl get services -n file-upload

# Port forward to test
kubectl port-forward -n file-upload svc/upload-service 8080:80

# Test the service
curl http://localhost:8080/health
curl -X POST -F "file=@test.jpg" http://localhost:8080/upload
```

## 🧪 Testing

### Unit Tests
```bash
# Create test file
cat << EOF > internal/handlers/upload_test.go
package handlers

import (
   "testing"
   "net/http"
   "net/http/httptest"
)

func TestHealthCheck(t *testing.T) {
   req, err := http.NewRequest("GET", "/health", nil)
   if err != nil {
       t.Fatal(err)
   }

   rr := httptest.NewRecorder()
   handler := http.HandlerFunc(HealthCheck)

   handler.ServeHTTP(rr, req)

   if status := rr.Code; status != http.StatusOK {
       t.Errorf("handler returned wrong status code: got %v want %v", status, http.StatusOK)
   }
}
EOF

# Run tests
go test ./...
```

## 📊 Monitoring

### Health Checks
- `/health` - Service health status
- `/files` - List uploaded files

### MinIO Console
- Access MinIO console at `http://localhost:9001`
- Username: `minioadmin`
- Password: `minioadmin`

## 🧹 Cleanup

### Docker Cleanup
```bash
cd docker
docker-compose down -v
```

### Kubernetes Cleanup
```bash
kubectl delete namespace file-upload
```

## 🎯 Learning Objectives
- File upload handling in containers
- Object storage integration
- Persistent volume management
- File type validation and security
- Multi-service communication
- Storage orchestration

## 📚 Key Takeaways
- **File Handling**: Secure file upload with validation
- **Object Storage**: MinIO for scalable file storage
- **Persistent Storage**: PVCs for data persistence
- **Security**: File type validation and size limits
- **Multi-Service**: Service communication patterns
- **Storage Management**: Object storage vs file storage

---

**Happy Learning! 📁🐳☸️** 