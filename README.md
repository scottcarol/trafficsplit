# trafficsplit
Use Buoyant's Booksapp to demo Traffic Split

1. Install the MySQL backend

    ```bash
    kubectl apply -f mysql-backend.yml
    ```

2. Verify that the mysql-init job successfully completes

    ```bash
    kubectl get po
    NAME                    READY     STATUS      RESTARTS   AGE
    mysql-9bd5bcfdf-7jb2s   1/1       Running     0          3m
    mysql-init-29nxv        0/1       Completed   0          3m
    ```

3. Install Linkerd and install the app configured to use MySQL

    ```bash
    linkerd install | kubectl apply -f -
    linkerd inject mysql-app.yml | kubectl apply -f -
    ```

4. Verify that the app works

    ```bash
    kubectl port-forward svc/webapp 7000
    open "http://localhost:7000"
    ```

5. Apply a clone of the authors service with accompanying deployment

    ```bash
    linkerd inject authors-clone.yml | kubectl apply -f -
    ```

6. Verify that the `authors-clone` deployment has been created and view metrics for both `authors` and `authors-clone`. Although `authors-clone` is not receiving traffic, its RPS will be `0.1` due to healthchecks and/or Prometheus queries. 

    ```bash
    watch linkerd stat deploy authors authors-clone
    NAME            MESHED   SUCCESS      RPS   LATENCY_P50   LATENCY_P95   LATENCY_P99   TCP_CONN
    authors            1/1    77.74%   4.9rps           3ms          19ms          43ms          4
    authors-clone      1/1   100.00%   0.1rps           4ms           4ms           4ms          1
    ```

7. In a new window, install a traffic split for the authors service to split the traffic 50/50 between `authors` and `authors-clone`.

    ```bash
    kubectl apply -f traffic-split.yml
    ```

8. Back at your `watch linkerd stat` window, confirm that the RPS for `authors` and `authors-clone` deployments are changing.

9. Go forth and split!
