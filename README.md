This `Vagrantfile` creates a Fedora 33 VM with a static IP address of 192.168.121.100.

To bring up a new server:

```
vagrant up
```

To log in with SSH:

```
vagrant ssh
```

Destroy the VM (to wipe everything and start over):

```
vagrant destroy
```

## Prerequisites

On the Vagrant VM, we will run Grafana and Prometheus in containers. To run those container images, we need to install podman. (Vim is just nice to have.)

Inside the Vagrant VM:

```
sudo yum -y install vim podman
```

## Prometheus

Prometheus requires a configuration file. Make the directory here:

```
sudo mkdir /etc/prometheus
```

Then open the file for editing:
```
sudo vim /etc/prometheus/prometheus.yml
```

Next, add the following contents to `/etc/prometheus/prometheus.yml`:

```yaml
# Global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  scrape_timeout: 15s  # scrape_timeout is set to the global default (10s).

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
    - targets: ['localhost:9090']
```

Save and quit Vim.

Lastly, we can run Prometheus like so. Use the unprivileged "vagrant" user account (no sudo):

```
podman run \
    -p 9090:9090 \
    --detach=true \
    -v /etc/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
    docker.io/prom/prometheus:latest
```

This command will pull the latest Prometheus image down from Docker Hub
("docker.io") and run it.

If podman worked, the last log message should be `msg="Server is ready to
receive web requests."`

Prometheus runs on TCP port 9090, and the `-p 9090:9090` option means "expose port 9090 from the container to my VM."

Next figure out VM IP. Should be hard-coded in the Vagrantfile. Mine
is 192.168.121.100.

To test that this is working, `curl` should print some small HTML on port 9090:

```
curl 192.168.121.100:9090
```

If curl says `Connection refused`, the container is not running.

```
curl: (7) Failed to connect to 192.168.121.100 port 9090: Connection refused
```

Open http://192.168.121.100:9090/graph in the browser. Prometheus has a simple web UI shown here.

Prometheus exposes metrics about itself: http://192.168.121.100:9090/metrics,
so you can use Prometheus to monitor Prometheus (inception-style). 

Prometheus "exports" data about itself. But in a real environment Prometheus
would load ["exporters" from many other sources of
data](https://prometheus.io/docs/instrumenting/exporters/), like web servers,
or database servers (Postgres), etc.

## Grafana

To download and run Grafana from Docker Hub:

```
podman run -d -p 3000:3000 --name grafana docker.io/grafana/grafana:6.5.0
```

This is really similar to the Prometheus example, except there is no
configuration file in `/etc`.

To test that this is working, `curl` should print some small HTML on port 3000:

```
curl 192.168.121.100:3000
```

You can open http://192.168.121.100:3000 in a browser. The default Grafana login is "admin/admin".

Click "Add Data Source" -> "Prometheus"
