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

Have you changed something in `Vagrantfile` after you've run `vagrant up`? Use
this to apply your new settings (and reboot the VM):

```
vagrant reload
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
    --net=host \
    --detach=true \
    --name prometheus \
    -v /etc/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
    docker.io/prom/prometheus:latest
```

This command will pull the latest Prometheus image down from Docker Hub
("docker.io") and run it.

You can view the logs with `podman logs`:

```
podman logs prometheus
```

If everything worked, the last log message should be `msg="Server is ready to
receive web requests."`

Prometheus runs on TCP port 9090, and the `--net=host` option means "expose this container's network to my VM".

*Networking notes*: The VM IP address on Linux (with libvirt) is is 192.168.121.100. On macos (with virtualbox), the VM IP is a private IP (10.0.x.x) and the host cannot access that IP directly. Instead, Vagrant forwards TCP ports from the Virtualbox VM to the host on 127.0.0.1.

To test that this is working, `curl` should print some small HTML on port 9090.

* On Linux host, connect to the VM's IP: `curl 192.168.121.100:9090`
* On macos host, connect to the forwarded port: `curl 127.0.0.1:9090`

If curl says `Connection refused`, the container is not running or there is another network problem.

```
curl: (7) Failed to connect to 127.0.0.1 port 9090: Connection refused
```

Open http://127.0.0.1:9090/graph in the browser. Prometheus has a simple web UI shown here.

Prometheus exposes metrics about itself: http://127.0.0.1:9090/metrics,
so you can use Prometheus to monitor Prometheus (inception-style). 

Prometheus "exports" data about itself. But in a real environment Prometheus
would load ["exporters" from many other sources of
data](https://prometheus.io/docs/instrumenting/exporters/), like web servers,
or database servers (Postgres), etc.

## Grafana

To download and run Grafana from Docker Hub:

```
podman run -d --net=host --name grafana docker.io/grafana/grafana:6.5.0
```

This is really similar to the Prometheus example, except there is no
configuration file in `/etc`.

To test that this is working, `curl` should print some small HTML on port 3000.

* On Linux host, connect to the VM's IP: `curl 192.168.121.100:3000`
* On macos host, connect to the forwarded port: `curl 127.0.0.1:3000`

You can open http://127.0.0.1:3000 in a browser. The default Grafana login is "admin/admin".

Click "Add Data Source" -> "Prometheus"
