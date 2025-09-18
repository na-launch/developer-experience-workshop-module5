# 🚀 **Module 5: Using Red Hat build of OpenTelemetry with Micrometer**

**Technology Stack:**

- Quarkus
- OpenTelemetry
- Micrometer

---

## 🎯 **Scenario**

Inside your workspace is a Java application that sends responses.  Your end users have been complaining about the application throwing intermittent errors.  The goal of this exercise is to use telemetry through the <a href="https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html/red_hat_build_of_opentelemetry" target="_blank">Red Hat build of OpenTelemetry</a> and  <a href="https://quarkus.io/guides/telemetry-micrometer-to-opentelemetry" target="_blank">Micrometer and OpenTelemetry bridge</a> to identify and fix issues in your application code.

* We will explore the Kubernetes Sidecar design pattern, which deploys an agent sidecar container within an application pod that sends traces to a centralized Grafana Tempo.
* The application will send tracing data to a *collector agent* (sidecar), which offloads responsibility by forwarding the data to a storage backend—in this case, the central Grafana Tempo instance.
* We will update the data pipeline in a later step to also capture and offload Micrometer metrics through the sidecar to Grafana Tempo.
* We will use Jaeger, the web console for Grafana Tempo, to analyze tracing spans and troubleshoot our application's issue.

---

## 🧩 **Challenge**

1. Launch Dev Spaces with the Git repository above.

2. From the Terminal, set up the environment by generating helper scripts:

  ```sh
  ./setup-env.sh
  ```

3. Run the workshop environment validation script.  You can ignore check #5 (quarkus CLI) which is optional.  Check #7 will flag as not passed which is to be expected since we haven't started deployment yet.

  ```sh
  ./workshop.sh    ## familiarize yourself with the helper script by viewing the subcommands
  ./workshop.sh check
  ```

4. Deploy the base application with the following helper script.  This step will take a couple of minutes:

  ```sh
  ./workshop.sh deploy
  ### OR ###
  mvn clean package -DskipTests
  ```

  - 4.1. You can validate a successful deployment with the following command:

  ```sh
  oc get pods
  NAME                                 READY   STATUS      RESTARTS   AGE
  micrometer-module-1-build            0/1     Completed   0          3m44s
  micrometer-module-79f458d8f9-mvxxj   1/1     Running     0          55s
  ```

5. Now we'll instrument our application.  Apply the OpenTelemetryCollector sidecar, ServiceMonitor, service, and route with:
  ```sh
  ./workshop.sh components
  ### OR ###
  oc apply -f resources/otel
  ```

6. In the following file `src/main/resources/application.properties`, update and save the following property with your username.

  **This will be the service name in Jaeger and must be unique so that you can find your application's telemetry (and not the rest of the classroom's!)**

  ```java
  # quarkus.otel.service.name=micrometer-module-{oc whoami}
  quarkus.otel.service.name=micrometer-module-replaceme
  ```

7. Redeploy the application with the helper script.  This step will take a couple of minutes to deploy, you can save some time by going to the next step in a separate browser tab.

  ```sh
  ./workshop.sh deploy
  
  oc get pods
  NAME                                 READY   STATUS      RESTARTS   AGE
  micrometer-module-1-build            0/1     Completed   0          7m47s
  micrometer-module-2-build            0/1     Completed   0          4m49s
  micrometer-module-79f458d8f9-mvxxj   2/2     Running     0          1m5s
  ```

8. Now we'll troubleshoot with telemetry data.  Navigate to <a href="https://tempo-tempo-stack-query-frontend-opentelemetry.apps.<cluster-domain>" target="_blank">Jaeger UI</a> (you can also get there from the OpenShift web console's application launch menu or Developer Hub homepage):
  - 8.1. Select your service which was defined in step #6 (i.e. micrometer-module-replaceme), set the lookback to the last 5 minutes, and press Find Traces
  - 8.2. Analyze multiple spans (both successful and error) taking note of associated tags and logs for spans generating errors.

9. We'll continue investigating by using Micrometer.  For our application, we used a Micrometer and OpenTelemetry bridge (quarkus-micrometer-opentelemetry) which puts metrics and logging telemetry all into the same data pipeline.  The main service (WorkResource.java) has been instrumented but the downstream service (DownstreamService.java) has not been.
  - 9.1. Navigate to `src/main/java/com/training/DownstreamService.java` in the source code and uncomment the imports below each `//TASK:`.  This will instrument the downstream service to collect more data to debug your application's issue.
  - 9.2. Redeploy the application with the helper script again.

  ```sh
  ./workshop.sh deploy
  
  oc get pods
  
  micrometer-module-1-build            0/1     Completed   0          11m27s
  micrometer-module-2-build            0/1     Completed   0          8m29s
  micrometer-module-3-build            0/1     Completed   0          1m24s
  micrometer-module-79f458d8f9-mvxxj   2/2     Running     0          52s
  ```

10. Back in the Jaeger web console, rerun your traces and observe the trace timeline consisting of additional spans that are created for the same workflow.  This may take a few minutes to generate the newly instrumented spans (in addition to the previous deployment's spans).

11. Micrometer has been integrated into OpenShift with the ServiceMonitor we deployed.  Navigate to <a href="https://console-openshift-console.apps.<cluster-domain>" target="_blank">OpenShift web console</a> -> Observe -> Metrics and search for the following metric:

  ```
  app_work_attempt_percentile_milliseconds
  ```

  Note the method and percentile of the individual metrics data.

12. Use both the telemetry data in Jaeger and Micrometer metrics to update your application so that it no longer produces spans with errors.

---

## 🥚 **Easter Eggs!**

- [ ] There are two easter eggs in Jaeger
  - As a hint, you will need to modify the application code to uncover them

---

## ✅ **Key Takeaways**

- Instrumented an application with Red Hat build of OpenTelemetry to collect tracing data  
- Further instrumented the application with Micrometer and OpenTelemetry bridge to send Micrometer metrics to OpenTelemetry
- Used tracing data in Jaeger web console for Grafana Tempo to troubleshoot the application
