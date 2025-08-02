# Lab 08: Jobs and CronJobs

## 🎯 Learning Objectives
- Understand Kubernetes Jobs for batch processing
- Configure CronJobs for scheduled tasks
- Manage job parallelism and completion
- Handle job failures and retries
- Learn job monitoring and cleanup

## ⏱️ Estimated Time: 60 minutes

## 📋 Prerequisites
- Kubernetes cluster (Minikube, kind, or cloud provider)
- kubectl configured and working
- Basic understanding of pods and deployments

## 🚀 Lab Exercises

### Exercise 1: Understanding Jobs

1. **Check existing Jobs:**
   ```bash
   kubectl get jobs --all-namespaces
   kubectl get jobs
   ```

2. **Create a namespace for Job testing:**
   ```bash
   kubectl create namespace job-demo
   kubectl config set-context --current --namespace=job-demo
   ```

### Exercise 2: Creating Basic Jobs

1. **Create a simple Job:**
   ```bash
   cat << EOF > simple-job.yaml
   apiVersion: batch/v1
   kind: Job
   metadata:
     name: simple-job
   spec:
     template:
       spec:
         containers:
         - name: simple-job
           image: busybox:1.35
           command: ['sh', '-c', 'echo "Hello from Job!" && date && sleep 10']
         restartPolicy: Never
     backoffLimit: 4
   EOF
   
   kubectl apply -f simple-job.yaml
   ```

2. **Monitor the Job:**
   ```bash
   kubectl get jobs
   kubectl get pods -l job-name=simple-job
   kubectl logs -l job-name=simple-job
   ```

3. **Check Job status:**
   ```bash
   kubectl describe job simple-job
   kubectl get job simple-job -o yaml
   ```

### Exercise 3: Job with Parallelism

1. **Create a Job with multiple completions:**
   ```bash
   cat << EOF > parallel-job.yaml
   apiVersion: batch/v1
   kind: Job
   metadata:
     name: parallel-job
   spec:
     completions: 5
     parallelism: 3
     template:
       spec:
         containers:
         - name: parallel-job
           image: busybox:1.35
           command: ['sh', '-c', 'echo "Processing task $JOB_COMPLETION_INDEX" && sleep 5']
         restartPolicy: Never
     backoffLimit: 4
   EOF
   
   kubectl apply -f parallel-job.yaml
   ```

2. **Monitor parallel execution:**
   ```bash
   kubectl get jobs parallel-job
   kubectl get pods -l job-name=parallel-job
   kubectl logs -l job-name=parallel-job
   ```

3. **Scale the Job:**
   ```bash
   # Increase parallelism
   kubectl scale job parallel-job --replicas=5
   
   # Check the scaling
   kubectl get pods -l job-name=parallel-job
   ```

### Exercise 4: Job with Resource Limits

1. **Create a Job with resource constraints:**
   ```bash
   cat << EOF > resource-job.yaml
   apiVersion: batch/v1
   kind: Job
   metadata:
     name: resource-job
   spec:
     template:
       spec:
         containers:
         - name: resource-job
           image: busybox:1.35
           command: ['sh', '-c', 'echo "Starting resource-intensive task" && dd if=/dev/zero of=/tmp/test bs=1M count=100 && echo "Task completed"']
           resources:
             requests:
               memory: "64Mi"
               cpu: "250m"
             limits:
               memory: "128Mi"
               cpu: "500m"
         restartPolicy: Never
     backoffLimit: 3
   EOF
   
   kubectl apply -f resource-job.yaml
   ```

2. **Monitor resource usage:**
   ```bash
   kubectl get pods -l job-name=resource-job
   kubectl describe pod -l job-name=resource-job
   kubectl logs -l job-name=resource-job
   ```

### Exercise 5: Job with ConfigMaps and Volumes

1. **Create a ConfigMap for the Job:**
   ```bash
   cat << EOF > job-config.yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: job-config
   data:
     script.sh: |
       #!/bin/sh
       echo "Processing data from $(hostname)"
       echo "Config value: $CONFIG_VALUE"
       echo "Input data: $(cat /data/input.txt)"
       echo "Output: $(date) - Processed by $(hostname)" > /data/output.txt
       echo "Job completed successfully"
   EOF
   
   kubectl apply -f job-config.yaml
   ```

2. **Create a Job with ConfigMap and volumes:**
   ```bash
   cat << EOF > config-job.yaml
   apiVersion: batch/v1
   kind: Job
   metadata:
     name: config-job
   spec:
     template:
       spec:
         containers:
         - name: config-job
           image: busybox:1.35
           command: ['sh', '-c', 'chmod +x /config/script.sh && /config/script.sh']
           env:
           - name: CONFIG_VALUE
             value: "test-config"
           volumeMounts:
           - name: config-volume
             mountPath: /config
           - name: data-volume
             mountPath: /data
         volumes:
         - name: config-volume
           configMap:
             name: job-config
         - name: data-volume
           emptyDir: {}
         restartPolicy: Never
     backoffLimit: 3
   EOF
   
   kubectl apply -f config-job.yaml
   ```

3. **Test the Job:**
   ```bash
   kubectl get pods -l job-name=config-job
   kubectl logs -l job-name=config-job
   ```

### Exercise 6: Understanding CronJobs

1. **Check existing CronJobs:**
   ```bash
   kubectl get cronjobs --all-namespaces
   kubectl get cronjobs
   ```

2. **Create a simple CronJob:**
   ```bash
   cat << EOF > simple-cronjob.yaml
   apiVersion: batch/v1
   kind: CronJob
   metadata:
     name: simple-cronjob
   spec:
     schedule: "*/2 * * * *"
     jobTemplate:
       spec:
         template:
           spec:
             containers:
             - name: simple-cronjob
               image: busybox:1.35
               command: ['sh', '-c', 'echo "CronJob executed at $(date)" && echo "Hello from CronJob!"']
             restartPolicy: OnFailure
         backoffLimit: 4
   EOF
   
   kubectl apply -f simple-cronjob.yaml
   ```

3. **Monitor the CronJob:**
   ```bash
   kubectl get cronjobs
   kubectl get jobs -l job-name=simple-cronjob
   kubectl get pods -l job-name=simple-cronjob
   ```

### Exercise 7: CronJob with Advanced Features

1. **Create a CronJob with concurrency policy:**
   ```bash
   cat << EOF > advanced-cronjob.yaml
   apiVersion: batch/v1
   kind: CronJob
   metadata:
     name: advanced-cronjob
   spec:
     schedule: "*/3 * * * *"
     concurrencyPolicy: Forbid
     successfulJobsHistoryLimit: 3
     failedJobsHistoryLimit: 1
     jobTemplate:
       spec:
         template:
           spec:
             containers:
             - name: advanced-cronjob
               image: busybox:1.35
               command: ['sh', '-c', 'echo "Advanced CronJob at $(date)" && sleep 30 && echo "Task completed"']
             restartPolicy: OnFailure
         backoffLimit: 2
   EOF
   
   kubectl apply -f advanced-cronjob.yaml
   ```

2. **Test concurrency policy:**
   ```bash
   # Check CronJob status
   kubectl get cronjobs
   kubectl describe cronjob advanced-cronjob
   
   # Manually trigger a job
   kubectl create job --from=cronjob/advanced-cronjob manual-trigger
   ```

### Exercise 8: CronJob with Resource Management

1. **Create a CronJob with resource limits:**
   ```bash
   cat << EOF > resource-cronjob.yaml
   apiVersion: batch/v1
   kind: CronJob
   metadata:
     name: resource-cronjob
   spec:
     schedule: "*/5 * * * *"
     concurrencyPolicy: Allow
     jobTemplate:
       spec:
         template:
           spec:
             containers:
             - name: resource-cronjob
               image: busybox:1.35
               command: ['sh', '-c', 'echo "Resource-intensive task at $(date)" && dd if=/dev/zero of=/tmp/test bs=1M count=50 && echo "Task completed"']
               resources:
                 requests:
                   memory: "32Mi"
                   cpu: "100m"
                 limits:
                   memory: "64Mi"
                   cpu: "200m"
             restartPolicy: OnFailure
         backoffLimit: 3
   EOF
   
   kubectl apply -f resource-cronjob.yaml
   ```

2. **Monitor resource usage:**
   ```bash
   kubectl get cronjobs
   kubectl get jobs -l job-name=resource-cronjob
   kubectl get pods -l job-name=resource-cronjob
   ```

### Exercise 9: Job and CronJob Monitoring

1. **Check Job status:**
   ```bash
   kubectl get jobs -o wide
   kubectl describe job simple-job
   kubectl get pods -l job-name=simple-job
   ```

2. **Check CronJob status:**
   ```bash
   kubectl get cronjobs -o wide
   kubectl describe cronjob simple-cronjob
   kubectl get jobs -l job-name=simple-cronjob
   ```

3. **Monitor events:**
   ```bash
   kubectl get events --sort-by='.lastTimestamp' | grep -i job
   kubectl get events --sort-by='.lastTimestamp' | grep -i cronjob
   ```

### Exercise 10: Job and CronJob Troubleshooting

1. **Common Job issues:**
   ```bash
   # Check Job status
   kubectl get jobs -o wide
   
   # Check pod status
   kubectl get pods -l job-name=simple-job
   
   # Check pod logs
   kubectl logs -l job-name=simple-job
   
   # Check pod events
   kubectl describe pod -l job-name=simple-job
   ```

2. **Common CronJob issues:**
   ```bash
   # Check CronJob status
   kubectl get cronjobs -o wide
   
   # Check Job status
   kubectl get jobs -l job-name=simple-cronjob
   
   # Check pod status
   kubectl get pods -l job-name=simple-cronjob
   
   # Check CronJob events
   kubectl describe cronjob simple-cronjob
   ```

3. **Debug commands:**
   ```bash
   # Check Job configuration
   kubectl get job simple-job -o yaml
   
   # Check CronJob configuration
   kubectl get cronjob simple-cronjob -o yaml
   
   # Check pod logs
   kubectl logs <pod-name>
   
   # Check pod events
   kubectl describe pod <pod-name>
   ```

### Exercise 11: Job and CronJob Cleanup

1. **Suspend a CronJob:**
   ```bash
   kubectl patch cronjob simple-cronjob -p '{"spec":{"suspend":true}}'
   
   # Check the status
   kubectl get cronjobs
   ```

2. **Resume a CronJob:**
   ```bash
   kubectl patch cronjob simple-cronjob -p '{"spec":{"suspend":false}}'
   
   # Check the status
   kubectl get cronjobs
   ```

3. **Delete completed Jobs:**
   ```bash
   # Delete specific Job
   kubectl delete job simple-job
   
   # Delete all Jobs in namespace
   kubectl delete jobs --all
   ```

## 🧹 Cleanup

1. **Delete Jobs:**
   ```bash
   kubectl delete -f simple-job.yaml
   kubectl delete -f parallel-job.yaml
   kubectl delete -f resource-job.yaml
   kubectl delete -f config-job.yaml
   ```

2. **Delete CronJobs:**
   ```bash
   kubectl delete -f simple-cronjob.yaml
   kubectl delete -f advanced-cronjob.yaml
   kubectl delete -f resource-cronjob.yaml
   ```

3. **Delete ConfigMap:**
   ```bash
   kubectl delete -f job-config.yaml
   ```

4. **Delete namespace:**
   ```bash
   kubectl delete namespace job-demo
   kubectl config set-context --current --namespace=default
   ```

5. **Clean up files:**
   ```bash
   rm -f *.yaml
   ```

## 📚 Key Takeaways

- **Jobs**: Run batch processes to completion
- **CronJobs**: Schedule Jobs to run at specific times
- **Parallelism**: Control how many pods run simultaneously
- **Completions**: Specify how many successful completions are needed
- **Concurrency Policy**: Control how CronJobs handle overlapping executions
- **Resource Management**: Set limits and requests for Job resources

## 🔍 Troubleshooting

### Common Issues:
1. **Job not completing**: Check pod logs and restart policy
2. **CronJob not scheduling**: Check schedule format and suspend status
3. **Resource issues**: Verify resource limits and node capacity
4. **Concurrency conflicts**: Check concurrency policy settings

### Debug Commands:
```bash
# Check Job status
kubectl get jobs -o wide

# Check CronJob status
kubectl get cronjobs -o wide

# Check pod status
kubectl get pods -l job-name=<job-name>

# Check logs
kubectl logs -l job-name=<job-name>

# Check events
kubectl get events --sort-by='.lastTimestamp'
```

## 🎯 Next Steps

- Learn about resource management and autoscaling
- Explore security and RBAC
- Study monitoring and observability
- Practice with production Job and CronJob patterns 