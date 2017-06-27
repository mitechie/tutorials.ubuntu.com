---

id: juju-controller-prometheus
summary: Learn how to properly monitor your production Juju controllers.
categories: juju
tags: controller, juju, prometheus, intermediate, grafana, telegraf, operations, infrastructure
difficulty: 5
status: Draft
Published: 2017-07-28

---


# Monitoring your production Juju Controllers


## Overview
Duration: 0:00

### Why is it important to monitor your Juju controllers.

Juju is a state driven system. There are agents running on each unit in each model that Juju controls and needs to be monitored to make sure that the Juju controllers are able to handle the load of the events going through the system. If the controllers are not able to keep up for any reason you might need to scale up the hardware your controllers run on, you might need to [migrate load inducing models](https://jujucharms.com/docs/models-migrate) to another set of controllers, or there could be a sign of something going wrong in running models that needs an operator to address.


### In this tutorial you’ll learn how to...

- Create an HA Juju Controller
- Enable collecting of hardware metrics from the controller machines
- Enable metrics collection from Juju controllers
- Setting up a graphical dashboard of important metrics to watch

### You will need...

* Access to several machines in a public cloud, private cloud, or other substrate that Juju supports


## Setting up an HA controller
Duration: 0:00

Setting up a set of HA Juju controllers is done using the [juju enable-ha](https://jujucharms.com/docs/controllers-ha) command. Start with bootstrapping onto your substrate.


    juju bootstrap aws
    juju enable-ha


Switch to the controller model so that it's active for the rest of this
tutorial.


    juju switch controller


Wait for the controllers to settle monitoring the `juju status`


    watch -n 10 --color juju status --color


    Model       Controller     Cloud/Region   Version  SLA
    controller  aws-us-east-1  aws/us-east-1  2.2.1    unsupported

    App  Version  Status  Scale  Charm  Store  Rev  OS  Notes

    Unit  Workload  Agent  Machine  Public address  Ports  Message

    Machine  State    DNS             Inst id              Series  AZ          Message
    0        started  54.144.157.236  i-049aa4b4811d41dd4  xenial  us-east-1a  running
    1        started  54.82.0.181     i-002a6b39210048476  xenial  us-east-1b  running
    2        started  54.161.120.120  i-0bc8bd027f3d4d3bc  xenial  us-east-1c  running


## Enabling collection of hardware metrics from the controller machines

[Telegraf](https://jujucharms.com/telegraf) is used to collect hardware based
metrics, such as memory and CPU details, and provide a [Prometheus](https://jujucharms.com/u/prometheus-charmers/prometheus]) compatible
format to collect those metrics from the machines.


    juju deploy cs:telegraf


To use the charms to
collect data from the machines that the controller is on a relation must be
made. Juju does not support relations to machines and a deployed
[Ubuntu](https://jujucharms.com/ubuntu) to the machine to provide that
relation endpoint. This loads the charm but since the machine is already
provisioned and is running Ubuntu it's nearly a noop.


    juju deploy ubuntu --to 0
    juju add-unit ubuntu --to 1
    juju add-unit ubuntu --to 2


Once the Ubuntu charm is added a relation to Telegraf can be established
enabling the metrics output.


    juju relate telegraf:juju-info ubuntu:juju-info


The metrics are then collected by an instance of Prometheus.


    juju deploy cs:~prometheus-charmers/prometheus

Once the machine is up and available a cert needs to be sent to the machine
for use.


    juju scp ./controller-ca-cert prometheus/0:~/controller-ca.cert
    juju ssh prometheus/0
    sudo mv controller-ca.cert /var/snap/prometheus/current
    <ctrl-D>

    juju relate telegraf:prometheus-client prometheus:target


Prometheus needs to be configured to scrape the information from the
controllers. The telegraf information will come over via a relation, but
there's no relation for the Juju provided data. A sample yaml file is
[available here]().

Update the configuration with the IP addresses of the controller machines.
Once the file is updated set it as configuration on the Prometheus
application.


    juju config prometheus scrape-jobs=@scrape-jobs.yaml


## Setting up the graphical dashboard.

[Grafana](https://jujucharms.com/u/prometheus-charmers/grafana) is used as a
graphical front end for the Prometheus data. It enables building of charts to
visualize any of the data made available via Telegraf or Juju. Deploy Grafana
and relate it to Prometheus to set up the data source.


    juju deploy cs:~prometheus-charmers/grafana
    juju add-relation prometheus:grafana-source grafana:grafana-source


Note that the password for the dashboard is set via Juju config. Make sure to
make this unique to your deployment.


    juju config grafana admin_password="supersecret"


Finally, expose the Grafana dashboard so that the firewall rules are updated
to allow external access.


    juju expose grafana


Locate the IP address of the Grafana dashboard through `juju status`.


    juju status grafana


Open the url for your Grafana dashboard to

Now log into your Grafana url

1. http://$ipaddr:3000
2. username: admin
3. password: $whatever_you_set_in_above_config

And you can load the included dashboards that are provided. The first is for XXXXX and the second is better for watching after YYYYY.


@todo when do we want to use which dashboard
@todo the mongodb state config isn't in here yet




......................Not yet edited

## That’s all folks!
Duration: 1:00

Congratulations! You’ve made it!

By now you should have your Kubernetes cluster up and running.
![Kubernetes bundle](./images/kubernetes-bundle.png)


### Next steps

Now that you have your production cluster, you can put it to work:

* [The easy way to commoditise GPUs for Kubernetes](https://medium.com/intuitionmachine/how-we-commoditized-gpus-for-kubernetes-7131f3e9231f).
* [Build a transcoding platform in minutes](https://github.com/deis/workflow).
* [Transform your solution into a private PaaS](https://insights.ubuntu.com/2017/03/27/job-concurrency-in-kubernetes-lxd-cpu-pinning-to-the-rescue/).


### Further reading

* Learn more about the [Canonical Distribution of Kubernetes](https://jujucharms.com/canonical-kubernetes/) bundle.
* Discover [Kubernetes ](https://jujucharms.com/kubernetes).
* Get involved and connect with the [Kubernetes community](https://kubernetes.io/community/).
