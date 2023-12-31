We have a web server container running the nginx image. The access and error logs generated by the web server are not critical enough to be placed on a persistent volume. However, Nautilus developers need access to the last 24 hours of logs so that they can trace issues and bugs. Therefore, we need to ship the access and error logs for the web server to a log-aggregation service. Following the separation of concerns principle, we implement the Sidecar pattern by deploying a second container that ships the error and access logs from nginx. Nginx does one thing, and it does it well—serving web pages. The second container also specializes in its task—shipping logs. Since containers are running on the same Pod, we can use a shared emptyDir volume to read and write logs.


Create a pod named webserver.

Create an emptyDir volume shared-logs.

Create two containers from nginx and ubuntu images with latest tag only and remember to mention tag i.e nginx:latest, nginx container name should be nginx-container and ubuntu container name should be sidecar-container on webserver pod.

Add command on sidecar-container "sh","-c","while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done"

Mount the volume shared-logs on both containers at location /var/log/nginx, all containers should be up and running.


-----------------------

To achieve this setup, you can create a YAML file named `webserver-pod.yaml` with the following contents:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  volumes:
    - name: shared-logs
      emptyDir: {}

  containers:
    - name: nginx-container
      image: nginx:latest
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx

    - name: sidecar-container
      image: ubuntu:latest
      command: ["sh", "-c", "while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done"]
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx
```

To apply this configuration and create the pod, run the following command:

```bash
kubectl apply -f webserver-pod.yaml
```

This YAML configuration defines a pod named `webserver` with two containers: `nginx-container` and `sidecar-container`. Both containers share an `emptyDir` volume named `shared-logs`, which is mounted at `/var/log/nginx` in both containers.

The `nginx-container` runs the Nginx web server, and the `sidecar-container` runs an Ubuntu image with a command that continuously concatenates the contents of Nginx's access and error logs, simulating log shipping to a log-aggregation service. The logs are read from the `/var/log/nginx/access.log` and `/var/log/nginx/error.log` paths, and the sidecar container sleeps for 30 seconds between concatenations.

Please note that this example uses a simple method to demonstrate the concept of the Sidecar pattern for log shipping. In a real-world scenario, you might want to use more sophisticated log-aggregation solutions like Fluentd, Logstash, or a cloud-based logging service.



Verify the container
------------------

To confirm that the logs are being successfully shipped and read by the sidecar container, you can follow these steps:

1. **Access the Pod's Shell**:
   Start by accessing the shell of the `sidecar-container` within the `webserver` Pod. You can use the following command to do this:

   ```bash
   kubectl exec -it webserver -c sidecar-container -- /bin/sh
   ```

   This command opens a shell session within the `sidecar-container`.

2. **Monitor Logs**:
   Once you are inside the `sidecar-container`, you can monitor the logs by running the following command:

   ```bash
   cat /var/log/nginx/access.log
   ```

   This command will display the contents of the `access.log` file. You should see the access logs being printed on the console.

3. **Exit Shell**:
   When you are done inspecting the logs, you can exit the shell by typing:

   ```bash
   exit
   ```

