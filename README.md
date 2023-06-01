# prometheusandgrafanalabs

Building a Prometheus Server
Introduction
Prometheus is a powerful tool for monitoring your infrastructure. Getting started with Prometheus is as easy as setting up a single Prometheus server. There are multiple ways to install Prometheus. In this lab, you will have the opportunity to set up a Prometheus server by installing Prometheus from pre-compiled binaries made available by the Prometheus team.
Solution
Log in to the server using the credentials provided:
ssh cloud_user@<PUBLIC_IP_ADDRESS>
Note: When copying and pasting code into Vi from the lab guide, first enter :set paste (and then i to enter insert mode) to avoid adding unnecessary spaces and hashes.
Download and Install Prometheus
Create the prometheus user:

 sudo useradd -M -r -s /bin/false prometheus


Create the prometheus directories:

 sudo mkdir /etc/prometheus /var/lib/prometheus


Download the pre-compiled binaries:

 wget https://github.com/prometheus/prometheus/releases/download/v2.16.0/prometheus-2.16.0.linux-amd64.tar.gz


Extract the binaries:

 tar xzf prometheus-2.16.0.linux-amd64.tar.gz prometheus-2.16.0.linux-amd64/


Move the files from the downloaded archive to the appropriate locations, and set ownership on these files and directories to the prometheus user:

 sudo cp prometheus-2.16.0.linux-amd64/{prometheus,promtool} /usr/local/bin/

 sudo chown prometheus:prometheus /usr/local/bin/{prometheus,promtool}

 sudo cp -r prometheus-2.16.0.linux-amd64/{consoles,console_libraries} /etc/prometheus/

 sudo cp prometheus-2.16.0.linux-amd64/prometheus.yml /etc/prometheus/prometheus.yml

 sudo chown -R prometheus:prometheus /etc/prometheus

 sudo chown prometheus:prometheus /var/lib/prometheus


Run Prometheus in the foreground to make sure everything is set up correctly so far:

 prometheus --config.file=/etc/prometheus/prometheus.yml

 In the output, we should see a message stating, "Server is ready to receive web requests."


Press Ctrl+C to stop the process.


Configure Prometheus as a systemd Service
Create a systemd unit file for Prometheus:

 sudo vi /etc/systemd/system/prometheus.service


Define the Prometheus service in the unit file:

 [Unit] Description=Prometheus Time Series Collection and Processing Server Wants=network-online.target After=network-online.target [Service] User=prometheus Group=prometheus Type=simple ExecStart=/usr/local/bin/prometheus \ --config.file /etc/prometheus/prometheus.yml \ --storage.tsdb.path /var/lib/prometheus/ \ --web.console.templates=/etc/prometheus/consoles \ --web.console.libraries=/etc/prometheus/console_libraries [Install] WantedBy=multi-user.target


Save and exit the file by pressing Escape followed by wq!.


Make sure systemd picks up the changes we made:

 sudo systemctl daemon-reload


Start the Prometheus service:

 sudo systemctl start prometheus


Enable the Prometheus service so it will automatically start at boot:

 sudo systemctl enable prometheus


Verify the Prometheus service is healthy:

 sudo systemctl status prometheus

 We should see its state is active (running).


Press Ctrl+C to stop the process.


Make an HTTP request to Prometheus to verify it is able to respond:

 curl localhost:9090

 The result should be <a href="/graph">Found</a>.


In a new browser tab, access Prometheus by navigating to http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090 (replacing <PROMETHEUS_SERVER_PUBLIC_IP> with the IP listed on the lab page). We should then see the Prometheus expression browser.


Conclusion
Congratulations on successfully completing this hands-on lab!
Additional Resources
You have just started your new job at LimeDrop, a new startup that provides a subscription service that ships a weekly box of limes to customers. Apparently, people are really into limes at the moment, so the company is growing rapidly. The company has built a complex infrastructure running both on-premises and in the cloud.
Unfortunately, LimeDrop has not yet invested much time in setting up monitoring for the various components of their infrastructure. This makes it difficult to diagnose and anticipate problems. Your first task is to set up a Prometheus server that will monitor LimeDrop's infrastructure. An Ubuntu server has been made available to you for this purpose.
Download and install the Prometheus pre-compiled binaries, and then configure Prometheus as a systemd service and start it.
If you get stuck, feel free to check out the solution video, or the detailed instructions under each objective. Good luck!














Collecting Linux Server Metrics with Prometheus
Introduction
Prometheus uses a pull model to periodically scrape metric data from targets. In order to collect data, you will need to set up an entity that is able to provide metric data, and then configure the Prometheus server to scrape it. In this lab, you will walk through the process of configuring Prometheus monitoring for a Linux server.
Solution
Log in to the LimeDrop web server using the credentials provided on the lab page:
ssh cloud_user@<LIMEDROP_PUBLIC_IP_ADDRESS>
Note: When copying and pasting code into Vi from the lab guide, first enter :set paste (and then i to enter insert mode) to avoid adding unnecessary spaces and hashes.
Install and Configure Node Exporter on the LimeDrop Web Server
Create a user and group that will be used to run Node Exporter:

 [cloud_user@limedrop]$ sudo useradd -M -r -s /bin/false node_exporter


Download the Node Exporter binary:

 [cloud_user@limedrop]$ wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz


Extract the Node Exporter binary:

 [cloud_user@limedrop]$ tar xvfz node_exporter-0.18.1.linux-amd64.tar.gz


Copy the Node Exporter binary to the appropriate location:

 [cloud_user@limedrop]$ sudo cp node_exporter-0.18.1.linux-amd64/node_exporter /usr/local/bin/


Set ownership on the Node Exporter binary:

 [cloud_user@limedrop]$ sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter


Create a systemd unit file for Node Exporter:

 [cloud_user@limedrop]$ sudo vi /etc/systemd/system/node_exporter.service


Define the Node Exporter service in the unit file:

 [Unit] Description=Prometheus Node Exporter Wants=network-online.target After=network-online.target [Service] User=node_exporter Group=node_exporter Type=simple ExecStart=/usr/local/bin/node_exporter [Install] WantedBy=multi-user.target


Save and exit the file by pressing Escape followed by :wq!.


Make sure systemd picks up the changes we made:

 [cloud_user@limedrop]$ sudo systemctl daemon-reload


Start the node_exporter service:

 [cloud_user@limedrop]$ sudo systemctl start node_exporter


Enable the node_exporter service so it will automatically start at boot:

 [cloud_user@limedrop_web]$ sudo systemctl enable node_exporter


Test that your Node Exporter is working by making a request to it from localhost:

 [cloud_user@limedrop]$ curl localhost:9100/metrics

 You should see some metric data in response to this request.


Configure Prometheus to Scrape Metrics from the LimeDrop Web Server
Open a new terminal session.


Log in to the Prometheus server:

 ssh cloud_user@<PROMETHEUS_PUBLIC_IP_ADDRESS>


Edit the Prometheus config file:

 [cloud_user@prometheus]$ sudo vi /etc/prometheus/prometheus.yml


Locate the scrape_configs section and add the following beneath it (ensuring it's indented to align with the existing job_name section):

 ... - job_name: 'LimeDrop Web Server' static_configs: - targets: ['10.0.1.102:9100'] ...


Save and exit the file by pressing Escape followed by :wq!.


Restart Prometheus to load the new config:

 [cloud_user@prometheus]$ sudo systemctl restart prometheus


Navigate to the Prometheus expression browser in your web browser using the public IP address of your Prometheus server: <PROMETHEUS_SERVER_PUBLIC_IP>:9090.


In the expression field (the box at the top of the page), paste in the following query to verify you are able to get some metric data from the LimeDrop web server:

 node_filesystem_avail_bytes{job="LimeDrop Web Server"}


Click Execute. We should then see several results.


Conclusion
Congratulations on successfully completing this hands-on lab!


Additional Resources
You recently started a new job at LimeDrop, a subscription service that provides weekly shipments of limes to customers. Earlier, you set up a Prometheus server to help provide monitoring for the company's infrastructure. One Linux web server might benefit particularly from this monitoring, as it has recently had some stability problems.
Your task is to configure Prometheus to collect metric data from this Linux server. Install Node Exporter on the server, and then configure the Prometheus server to scrape metrics from that exporter.
If you get stuck, feel free to check out the solution video or the detailed instructions under each objective. Good luck!

























Locating Time Series Data in Prometheus
Introduction
Prometheus allows you to collect a wide variety of metric data in one location. It also provides useful tools for exploring and querying your data. In this lab, you will have the opportunity to perform some very basic queries to obtain some data from a Prometheus server. You will obtain the current value of a metric, and you will also look up a set of multiple time-series values for that metric.
Solution
Log in to the server using the credentials provided:
ssh cloud_user@<PUBLIC_IP_ADDRESS>
Get the Current Temperature of the Data Center
In a browser, access the Prometheus expression browser:

 http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090


Run a query to obtain the current datacenter_temp:

 datacenter_temp


Click Execute.


Copy the current temperature value.


In the Prometheus server terminal, open the file that will contain the temperature data:

 vi current_temp.txt


Enter or paste in the current temperature value obtained from the Prometheus expression browser.


Save and exit the file by pressing Escape followed by :wq.


Get All Time-Series Entries for the Data Center Temperature Over the Last Five Minutes
In the Prometheus expression browser, run a query to obtain the data:

 datacenter_temp[5m]


Click Execute.


Copy all the values (including the timestamps).


In the Prometheus server terminal, open the file that will contain the data:

 vi 5_minute_temp.txt


Paste in the time-series data obtained from the Prometheus expression browser.


Save and exit the file by pressing Escape followed by :wq.


Conclusion
Congratulations on successfully completing this hands-on lab!

Additional Resources
Your company, LimeDrop, has an on-site data center. They are hoping to move the data center's functionality to the cloud in the future, but in the meantime it needs to be kept up and running. Unfortunately, some of the cooling equipment has been having issues, leading to temperature fluctuations.
Last week, you set up a Prometheus server to provide a central place to collect metrics for LimeDrop's large infrastructure. One of your data center admins set up the Prometheus server to collect the current temperature of the data center from a digital thermometer system.
You have been asked to check on the current status of the data center temperature using metrics collected by Prometheus. Obtain the current temperature and save the value to a file on the server. Then, obtain all the time-series temperature values collected over the last five minutes and save them to a file as well.
Some additional details:
The data center temperature data is stored in a metric called datacenter_temp.
Save the current temperature to the file /home/cloud_user/current_temp.txt.
Save the temperature values for the last five minutes to the file /home/cloud_user/5_minute_temp.txt.






























Working with Prometheus Metric Types
Introduction
Prometheus exporters provide data using a handful of different metric types, each representing its own unique kind of data. When working with Prometheus, it is important to understand how these different metric types function in order to properly interpret data.
In this lab, you will have the opportunity to collect data from metrics that utilize a variety of different metric types. This will allow you to work with multiple metric types hands-on.
Solution
Log in to the server using the credentials provided:
ssh cloud_user@<PUBLIC_IP_ADDRESS>
Get the Value of the mobile_gateway_restarts Counter
Access the Prometheus expression browser in your web browser using the public IP address of the Prometheus server:

 http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090


Run a query to get the value of the mobile_gateway_restarts counter:

 mobile_gateway_restarts


Click Execute.


In the terminal, open the report file:

 vi report.md


Add the number of restarts to the appropriate line in the file for the mobile_gateway_restarts metric.


Get the Value of the mobile_gateway_mem_available Gauge
In the Prometheus expression browser, run a query to get the value of the mobile_gateway_mem_available gauge:

 mobile_gateway_mem_available


Click Execute.


In the terminal, add the current available memory to the appropriate line in the file for the mobile_gateway_mem_available metric.


Get the Requested Data from the mobile_gateway_request_duration_seconds Histogram
In the Prometheus expression browser, run a query to get the value of the mobile_gateway_request_duration_seconds bucket le="3":

 mobile_gateway_request_duration_seconds_bucket{le="3"}


Click Execute.


In the terminal, add the number of requests in the three-second bucket to the appropriate line in the file for the mobile_gateway_request_duration_seconds{le="3"} metric.


In the Prometheus expression browser, run a query to get the total count for the mobile_gateway_request_duration_seconds histogram:

 mobile_gateway_request_duration_seconds_count


Click Execute.


In the terminal, add the total count to the appropriate line in the file for the mobile_gateway_request_duration_seconds_count metric.


Conclusion
Congratulations on successfully completing this hands-on lab!


Additional Resources
Your company, LimeDrop, uses Prometheus to monitor a variety of systems. One of the applications monitored by Prometheus is a gateway API that supports the LimeDrop mobile app. The data from this application contains metrics that use several different metric types. You have been asked to provide some metric data about the application for later review. You will find a file on the Prometheus server located at /home/cloud_user/report.md. Collect the requested data and enter the values into that file to complete the report.
Locate the following metric data and save it in report.md:
Get the current value of the mobile_gateway_restarts counter. This counter measures the number of restarts for the backend gateway that supports the LimeDrop mobile app.
Get the current value of the mobile_gateway_mem_available gauge. This gauge measures the amount of free memory available to the application.
There is a histogram metric called mobile_gateway_request_duration_seconds that tracks HTTP requests to the application in buckets by their duration. Obtain the value for the number of requests in the le="3" bucket.
Get the _count value for the mobile_gateway_request_duration_seconds histogram.


Working with Prometheus Queries
Introduction
Once you have begun collecting metric data in Prometheus, you will need to be able to work with that data in order to obtain information you can act upon. Prometheus provides a specialized query language known as Prometheus Query Language (PromQL) that allows you to write both simple and complex queries in order to retrieve and view your metric data in a useful way. In this hands-on lab, you will have the opportunity to work with Prometheus queries by writing and executing some simple queries.
Solution
Log in to the server using the credentials provided:
ssh cloud_user@<PUBLIC_IP_ADDRESS>
Get the Current Memory Usage for the LimeDrop Authentication Service
Access the Prometheus expression browser in a web browser (replacing <PROMETHEUS_SERVER_PUBLIC_IP> with the public IP address of the Prometheus server):

 http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090


Run a query to obtain the available memory amount, supplying a label to search specifically for metrics for the limedrop-auth:4455 instance:

 mem_available_total{instance="limedrop-auth:4455"}


Click Execute.


Note down the returned value.


In the terminal, open the output file:

 vi auth_service_data.md


Enter the current available memory (the value you copied from the query) on the mem_available_total line:

 mem_available_total: <AVAILABLE_MEMORY_VALUE>


Get the Available Memory Range Data Over a Five-Minute Period Three Minutes Ago
Run a query in the expression browser to obtain the data:

 mem_available_total{instance="limedrop-auth:4455"}[5m] offset 3m


Click Execute.


Note down the output, including the timestamps.


On the Prometheus server, still in the auth_service_data.md file, paste in the data at the end of the file.


Save and exit the file by pressing Escape followed by :wq.


Conclusion
Congratulations on successfully completing this hands-on lab!
Additional Resources
Your company, LimeDrop, has a Prometheus instance that is used to monitor a variety of infrastructure components. One of these components Prometheus monitors is an authentication service. A few users have reported issues with authentication since the last deployment.
You have been asked to provide some data about the authentication service to the developers as part of the troubleshooting process. Use Prometheus queries to obtain the requested information and save it in a file on the Prometheus server located at /home/cloud_user/auth_service_data.md.
The developers suspect poor memory performance may possibly be causing the issues. Collect the current available memory using the metric mem_available_total. Note there may be multiple instances that have data associated with this metric name, so you will need to look for the specific instance called limedrop-auth:4455.
Information about the current available memory may not be enough to pinpoint the problem. The developers would also like a range of data from the time period when the issue last occurred. The issue just occurred again three minutes ago. Get the values for mem_available_total from the limedrop-auth:4455 instance, but this time get all of the values for a five-minute period starting three minutes ago.

Advanced Prometheus Queries
Introduction
The Prometheus Query Language (PromQL) provides a variety of tools that enable you to transform your raw metric data into useful and actionable information. In this lab, you will have the opportunity to explore some advanced features of Prometheus queries as you build queries to solve slightly complex problems.
Solution
Log in to the server using the credentials provided:
ssh cloud_user@<PUBLIC_IP_ADDRESS>
Write a Query to Determine Which Instances Have High CPU Utilization
Access the Prometheus expression browser in your web browser:

 http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090


Run a query to add the CPU usage in the system and user modes for each instance. Then, filter the results to only instances where the combined number of CPU seconds is more than 10000:

 (node_cpu_seconds_total{mode="system"} + ignoring(mode) node_cpu_seconds_total{mode="user"}) > 10000


Click Execute.


On the Prometheus server, open the output file:

 vi report.md


In the appropriate section, record the list of instance names from the results of the Prometheus query.


Get the Per-Second Rate of Increase in HTTP Request Duration for All Instances
Run a query from the Prometheus expression browser:

 rate(http_request_duration_seconds_sum[5m])


Click Execute.


Copy all of the output, including the element data and values.


In the output file, paste in the data obtained using the query at the end of the file.


Save and exit the file by pressing Escape followed by :wq.


Conclusion
Congratulations on successfully completing this hands-on lab!

Additional Resources
Your company, LimeDrop, has a Prometheus instance that monitors a variety of applications. Some user reports have come in indicating there may be some performance issues somewhere in the company's infrastructure. Unfortunately, these reports do not include enough information to pinpoint the problem.
You have been asked to collect some specific data points from Prometheus in order to help locate the issue. Write and execute queries to obtain the requested data, then record it in a file on the Prometheus server located at /home/cloud_user/report.md.
The team performing the troubleshooting needs to know which instances have high CPU usage. Write a query that will add together CPU usage in seconds in both the system and user modes on each instance. Then, add a filter to the query that will return only records where the combined system and user seconds exceed 10000. You can use the node_cpu_seconds_total metric to get this information. Save the instance names of the instances exhibiting high usage to the output file.
The team wants to be able to determine how quickly HTTP request duration is currently increasing for all instances. Using the http_request_duration_seconds_sum metric, write a query to determine the rate of increase in HTTP request duration over the last five minutes. Save the resulting value to the output file.

Using the Prometheus HTTP API
Introduction
Tools such as the Prometheus expression browser can provide an easy way to execute queries and interact with Prometheus data. However, Prometheus also provides an HTTP API that can allow you to integrate Prometheus with your own custom tools. In this lesson, you will perform some simple queries using the Prometheus HTTP API, allowing you to gain hands-on experience with using that API.
Solution
Log in to the server using the credentials provided:
ssh cloud_user@<PUBLIC_IP_ADDRESS>
Get the Current Number of Threads
Make an HTTP request to the Prometheus server to retrieve the data and save it to a file:

 curl localhost:9090/api/v1/query?query=num_threads > /home/cloud_user/num_threads.txt


View the file to verify the output is there:

 cat /home/cloud_user/num_threads.txt


Get the num_threads Data Over the Last Five Minutes
Make an HTTP request to retrieve the data and save it to a file:

 start=$(date --date '-5 min' +'%Y-%m-%dT%H:%M:%SZ')

 end=$(date +'%Y-%m-%dT%H:%M:%SZ')

 curl "localhost:9090/api/v1/query_range?query=num_threads&start=$start&end=$end&step=15s" > /home/cloud_user/num_threads_5_minutes.txt


View the file to verify the output is there:

 cat /home/cloud_user/num_threads_5_minutes.txt


Conclusion
Congratulations on successfully completing this hands-on lab!


Additional Resources
Your company, LimeDrop, has a Prometheus server that collects metrics for a variety of applications and components. The developers would like to build some custom applications that pull data from Prometheus, but they are not sure how to do this.
You have been asked to pull some data from Prometheus using the HTTP API in order to demonstrate how this can be done. Use HTTP requests to execute the requested queries and save the resulting output to a file.
Get the current number of open threads in use by the LimeDrop web API. You can find this information by querying the num_threads metric. Save the result to a file located at /home/cloud_user/num_threads.txt.
Get the time-series data for the num_threads over the last five minutes. Note that the Prometheus HTTP API uses a separate endpoint for time-range queries. Save the result to a file located at /home/cloud_user/num_threads_5_minutes.txt.

Building a Prometheus Console Template
Introduction
Prometheus collects a wide variety of metric data. However, to make use of that data in the real world, you need a way to visualize important metrics at a glance. You can do this by building dashboards or similar pages that display real-time data in a useful format. One way to build these views is to use console templates. In this lab, you will have the opportunity to build a simple console template to display some statistics about a Linux server. This will give you some hands-on experience in working with console templates.
Solution
Log in to the server using the credentials provided:
ssh cloud_user@<PUBLIC_IP_ADDRESS>
Add the Server Status Metric to the Console Template
Create and edit the console template file:

 sudo vi /etc/prometheus/consoles/limedrop-web.html


Implement a display of the current "up" status of the web server:

 {{template "head" .}} {{template "prom_content_head" .}} <h1>LimeDrop Web Server Metrics</h1> <strong>Server Status:</strong> {{ template "prom_query_drilldown" (args "up{job='Web Server'}") }} {{template "prom_content_tail" .}} {{template "tail"}}


Save the file by pressing Escape followed by :w. Leave the file open.


View the console in a browser to verify it is working:

 http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090/consoles/limedrop-web.html


Add the CPU Usage Metric to the Console Template
In the console template file, implement a display for the web server's CPU usage:

 {{template "head" .}} {{template "prom_content_head" .}} <h1>LimeDrop Web Server Metrics</h1> <strong>Server Status:</strong> {{ template "prom_query_drilldown" (args "up{job='Web Server'}") }} <br /> <br /> <strong>Current CPU Usage:</strong> {{ template "prom_query_drilldown" (args "sum(rate(node_cpu_seconds_total{instance='limedrop-web:9100',mode!='idle'}[5m])) * 100 / 2" "%") }} {{template "prom_content_tail" .}} {{template "tail"}}


Save the file by pressing Escape followed by :w. Leave the file open.


Refresh the console in the browser to verify it is working.


Right-click the listed current CPU usage to open it in a new tab. This will take us to the expression browser with the query we used to calculate the metric.


Add the Memory Usage Metric to the Console Template
In the console template file, implement a display for the web server's memory usage:

 {{template "head" .}} {{template "prom_content_head" .}} <h1>LimeDrop Web Server Metrics</h1> <strong>Server Status:</strong> {{ template "prom_query_drilldown" (args "up{job='Web Server'}") }} <br /> <br /> <strong>Current CPU Usage:</strong> {{ template "prom_query_drilldown" (args "sum(rate(node_cpu_seconds_total{instance='limedrop-web:9100',mode!='idle'}[5m])) * 100 / 2" "%") }} <br /> <br /> <strong>Current Memory Usage:</strong> {{ template "prom_query_drilldown" (args "100 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100" "%") }} {{template "prom_content_tail" .}} {{template "tail"}}


Save and exit the file by pressing Escape followed by :wq.


Refresh the console in a browser to verify it is working.


Conclusion
Congratulations on successfully completing this hands-on lab!


Additional Resources
Your company, LimeDrop, is using Prometheus for monitoring. One of the things they are monitoring with Prometheus is a basic web server. This server occasionally encounters issues related to high CPU and/or memory load.
While it is possible to query Prometheus for the web server's memory and CPU usage data at any time, the administrators would like to have a dashboard they can access in order to quickly view the data at a glance.
Prometheus is already set up to monitor the web server with a job called Web Server. The web server's instance name is limedrop-web:9100.
Build a Prometheus console template to display the following pieces of information:
Current "up" status of the web server (whether or not Prometheus was able to successfully reach the server and scrape metrics).
Hint: Use the up metric.
Current percentage of CPU usage across all CPUs.
Hint: Use the node_cpu_seconds_total metric, keeping in mind the idle mode means the CPU is not in use.
Current percentage of memory usage.
Hint: The node_memory_MemAvailable_bytes and node_memory_MemTotal_bytes are useful here.
Implement your console template in a file located at /etc/prometheus/consoles/limedrop-web.html.
Once you have created the file, you should be able to test it by accessing it in a web browser at http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090/consoles/limedrop-web.html.

Using the Graph Library in a Prometheus Console Template
Introduction
Console templates provide a way to display real-time metric data in a useful format without the need for any external tools outside of Prometheus. With the console template graph library, you can even display graphs in order to visualize your data. In this lab, you will have the opportunity to implement some basic graphs within a console template. This will provide you with hands-on experience building graphs into your console templates.
Solution
Log in to the server using the credentials provided:
ssh cloud_user@<PUBLIC_IP_ADDRESS>
Build a Console Template with a Graph Displaying CPU Usage
Create and edit the console template file:

 sudo vi /etc/prometheus/consoles/limedrop-web.html


Implement a graph displaying CPU usage by editing the file to match the following:

 {{template "head" .}} {{template "prom_content_head" .}} <h1>LimeDrop Web Server Metrics</h1> <h3>CPU Usage</h3> <div id="cpuGraph"></div> <script> new PromConsole.Graph({ node: document.querySelector("#cpuGraph"), expr: "sum(rate(node_cpu_seconds_total{instance='limedrop-web:9100',mode!='idle'}[5m])) * 100 / 2" }) </script> {{template "prom_content_tail" .}} {{template "tail"}}


Save and exit the file by pressing Escape followed by :wq.


View the console in a browser to verify it is working, replacing <PROMETHEUS_SERVER_PUBLIC_IP> with the public IP of the Prometheus server listed on the lab page:

 http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090/consoles/limedrop-web.html


Build a Console Template with a Graph Displaying Memory Usage
Edit the console template file:

 sudo vi /etc/prometheus/consoles/limedrop-web.html


Implement a graph displaying memory usage by editing the file to match the following:

 {{template "head" .}} {{template "prom_content_head" .}} <h1>LimeDrop Web Server Metrics</h1> <h3>CPU Usage</h3> <div id="cpuGraph"></div> <script> new PromConsole.Graph({ node: document.querySelector("#cpuGraph"), expr: "sum(rate(node_cpu_seconds_total{instance='limedrop-web:9100',mode!='idle'}[5m])) * 100 / 2" }) </script> <h3>Memory Usage</h3> <div id="memGraph"></div> <script> new PromConsole.Graph({ node: document.querySelector("#memGraph"), expr: "100 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100" }) </script> {{template "prom_content_tail" .}} {{template "tail"}}


Save and exit the file by pressing Escape followed by :wq.


Refresh the console template in the browser, where we should see our new memory usage graph in addition to the CPU usage graph.


Conclusion
Congratulations on successfully completing this hands-on lab!


Additional Resources
Implement a console template in the file at /etc/prometheus/consoles/limedrop-web.html. Include two graphs, one for each of the following:
CPU usage over time. Here's a query you can use:
 sum(rate(node_cpu_seconds_total{instance='limedrop-web:9100',mode!='idle'}[5m])) * 100 / 2


Memory usage over time. Here's a query you can use:
 100 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100




Building a Grafana Instance to Work with Prometheus Data
Introduction
Prometheus is a powerful tool for reliably collecting and serving metric data, but its built-in visualization capabilities are limited. Luckily, Prometheus can easily integrate with more robust visualization tools such as Grafana, allowing you to build robust and useful dashboards to get the most out of your data. In this lab, you will have the opportunity to build a Grafana server and configure it to access Prometheus metric data. After completing this lab, you will know how to install Grafana and integrate it with Prometheus.
Solution
Log in to the Grafana server using the credentials provided:
ssh cloud_user@<GRAFANA_SERVER_PUBLIC_IP>
Install and Run Grafana
Log in to the Grafana server.


Install some required packages:

 sudo apt-get install -y apt-transport-https software-properties-common wget


Add the GPG key for the Grafana OSS repository:

 wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -


Add the repository:

 sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"


Install the Grafana package:

 sudo apt-get update

 sudo apt-get install grafana=6.6.2


Enable and start the grafana-server service:

 sudo systemctl enable grafana-server

 sudo systemctl start grafana-server


Make sure the service is in the active (running) state:

 sudo systemctl status grafana-server


Press Ctrl+C to stop the process.


Verify Grafana is working by accessing it in a web browser at http://<GRAFANA_SERVER_PUBLIC_IP>:3000.


Configure a Prometheus Data Source on the Grafana Server
Log in to Grafana with the username admin and password admin.


Reset the password when prompted.


Click Add data source.


Select Prometheus.


For the URL, enter http://10.0.1.101:9090. Note that 10.0.1.101 is the private IP address of the Prometheus server.


Click Save & Test. You should see a banner that says, Data source is working.


Click the Explore icon on the left.


In the PromQL Query input, enter a simple query, such as up.


Click Run Query. Some data should then appear. If so, congratulations! This data comes from the Prometheus server.


Conclusion
Congratulations on successfully completing this hands-on lab!


Additional Resources
Your company, LimeDrop, has a Prometheus server collecting metrics for a variety of applications and infrastructure components. Your admins have been able to query Prometheus for data as well as build a few useful console templates, but it has become clear that a more robust visualization tool is needed.
You will need to do the following:
Install Grafana on the provided Grafana server.
Configure Grafana to pull metric data from the Prometheus server.

Building a Prometheus Dashboard in Grafana
Introduction
Prometheus and Grafana create a powerful combination. Grafana allows you to build useful visualizations on top of your Prometheus metric data. In this lab, you will have the opportunity to build a Grafana dashboard to visualize Prometheus metrics. This will give you some hands-on experience with building useful Grafana dashboards on top of Prometheus.
Solution
Access the Grafana server at http://<GRAFANA_SERVER_PUBLIC_IP>:3000.


Log in with the username admin.


The default admin password is the same as the server password.


Create the Dashboard and Add a Web Server Status Panel
Click the Create button on the left, and then select Dashboard.


Click the Save Dashboard button near the top right.


For the dashboard name, enter "LimeDrop Web Server".


Click Save.


Click the Add panel button near the top right.


Click Add Query.


For the PromQL query, enter:

 up{instance="limedrop-web:9100"}


Click the Visualization icon.


Click the visualization type dropdown (which currently says Graph) and change it to Singlestat.


Under Value Mappings, enter two value to text mappings:


1 -> Up
0 -> Down
Click the General icon.


Change the panel title to "Server Status".


Click the back button in the top left (next to LimeDrop Web Server). You should see your dashboard, and the Server Status panel should say Up.


Click the Save Dashboard button near the top right, and then Save to save your changes.


Create a CPU Usage Graph Panel
Click the Add panel button near the top right.


Click Add Query.


For the PromQL query, enter:

 sum(rate(node_cpu_seconds_total{instance='limedrop-web:9100',mode!='idle'}[5m])) * 100


Click the General icon.


Change the panel title to "CPU Usage".


Click the back button in the top left. You should see your dashboard, and there should be a graph showing CPU utilization.


Click the Save Dashboard button near the top right, and then Save to save your changes.


Create a Memory Usage Graph Panel
Click the Add Panel button near the top right.


Click Add Query.


For the PromQL query, enter:

 100 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100


Click the General icon.


Change the panel title to "Memory Usage".


Click the back button in the top left. You should see your dashboard, and there should be a graph showing memory utilization.


Rearrange your panels by dragging and dropping them if desired.


Click the Save Dashboard button near the top right, and then Save to save your changes.


Conclusion
Congratulations on successfully completing this hands-on lab!

Additional Resources
Your company, LimeDrop, is using Prometheus and Grafana to monitor a variety of applications and infrastructure components. One of the servers being monitored is a Linux web server. The metrics for this server are in Prometheus, but there is currently no way for the administrators to visualize the data.
Your task is to build a dashboard in Grafana to display some basic information about the web server.
Build a dashboard with the title LimeDrop Web Server, and include the following:
A panel displaying the current status of the web server as "Up" or "Down". Use the up{instance="limedrop-web:9100"} metric.
A panel displaying a graph of CPU usage over time. Example query:
 sum(rate(node_cpu_seconds_total{instance='limedrop-web:9100',mode!='idle'}[5m])) * 100


A panel displaying a graph of Memory usage over time. Example query:
 100 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100


You can access the Grafana server at http://<GRAFANA_SERVER_PUBLIC_IP>:3000. Log in with the username admin. The default admin password is the same as the server password.

Collecting Application Metrics with Prometheus
Introduction
Prometheus exporters allow you to collect metric data from a variety of sources. You can monitor not only your servers but also the applications running on your servers. In this lab, you will have the opportunity to explore application monitoring in Prometheus by setting up monitoring for an application process and pulling application metric data into a Prometheus server.
Solution
Log in to the Apache server using the credentials provided:
ssh cloud_user@<APACHE_SERVER_PUBLIC_IP>
Install and Configure the Apache Exporter
Create a user for the Apache exporter:

 sudo useradd -M -r -s /bin/false apache_exporter


Download the Apache exporter binary:

 wget https://github.com/Lusitaniae/apache_exporter/releases/download/v0.7.0/apache_exporter-0.7.0.linux-amd64.tar.gz


Extract it:

 tar xvfz apache_exporter-0.7.0.linux-amd64.tar.gz


Copy it to /usr/local/bin/:

 sudo cp apache_exporter-0.7.0.linux-amd64/apache_exporter /usr/local/bin/


Change ownership:

 sudo chown apache_exporter:apache_exporter /usr/local/bin/apache_exporter


Create a systemd unit file for Apache exporter:

 sudo vi /etc/systemd/system/apache_exporter.service


Paste the following into the file:

 [Unit] Description=Prometheus Apache Exporter Wants=network-online.target After=network-online.target [Service] User=apache_exporter Group=apache_exporter Type=simple ExecStart=/usr/local/bin/apache_exporter [Install] WantedBy=multi-user.target


Save and exit the file by pressing Escape followed by :wq.


Start and enable the apache_exporter service:

 sudo systemctl enable apache_exporter

 sudo systemctl start apache_exporter


Make sure the service is running:

 sudo systemctl status apache_exporter

 We should see it's in the active (running) state.


Press q to quit the log output. 


Verify it is serving metrics:

 curl localhost:9117/metrics

 We should see a lot of data returned.


Configure Prometheus to Scrape Metrics from the Apache Exporter
Open a new terminal.


Log in to the Prometheus server:

 ssh cloud_user@<PROMETHEUS_SERVER_PUBLIC_IP>


Edit the Prometheus config:

 sudo vi /etc/prometheus/prometheus.yml


Under the scrape_configs section, add a scrape configuration for the Apache exporter:

 - job_name: 'Apache Server' static_configs: - targets: ['limedrop-apache:9117']


Save and exit the file by pressing Escape followed by :wq.


Restart Prometheus to load the new configuration:

 sudo systemctl restart prometheus


Access the expression browser in a web browser at http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090.


Run a query to view some Apache metric data:

 apache_workers

 We should see some data returned.


Conclusion
Congratulations on successfully completing this hands-on lab!


Additional Resources
Your company, LimeDrop, is using Prometheus for monitoring. They have an Apache server that is serving some static web content, and they would like to gather some metrics about the Apache application running on the server.
You can collect Apache metrics using the Apache exporter. Your task is to install and run Apache exporter on the Apache server, and then configure Prometheus to scrape Apache metrics.
Some additional info:
You can find documentation on the Apache exporter on the Apache Exporter GitHub page.
The Apache server is reachable from the Prometheus server at the hostname limedrop-apache.
If you are interested in checking out the LimeDrop static site, you can find it at http://<APACHE_SERVER_PUBLIC_IP>:80.

Docker Daemon Monitoring with Prometheus
Introduction
Prometheus is capable of monitoring a wide variety of systems and infrastructures, including Docker. In this lab, you will have the opportunity to set up Prometheus to monitor a Docker daemon. This will give you some hands-on experience with the process of monitoring Docker daemons with Prometheus.
Solution
Log in to the Docker server using the credentials provided:
ssh cloud_user@<DOCKER_SERVER_PUBLIC_IP>
Configure the Docker Daemon to Serve Prometheus Metrics
Create the Docker config file:

 sudo vi /etc/docker/daemon.json


Add configuration to enable experimental mode and set the metrics address:

 { "experimental": true, "metrics-addr": "10.0.1.102:9323" }


Save and exit the file by pressing Escape followed by :wq.


Restart Docker to load the new configuration:

 sudo systemctl restart docker


Verify Docker is serving Prometheus metrics:

 curl 10.0.1.102:9323/metrics

 We should see a lot of data returned.


Configure Prometheus to Scrape Docker Metrics
Open a new terminal.


Log in to the Prometheus server:

 ssh cloud_user@<PROMETHEUS_SERVER_PUBLIC_IP>


Edit the Prometheus config:

 sudo vi /etc/prometheus/prometheus.yml


Under the scrape_configs section, add a scrape configuration for Docker:

 - job_name: 'Docker' static_configs: - targets: ['limedrop-docker:9323']


Save and exit the file by pressing Escape followed by :wq.


Restart Prometheus to load the new configuration:

 sudo systemctl restart prometheus


Access the expression browser in a web browser at http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090.


Run a query to view some Docker metric data:

 engine_daemon_container_states_containers

 We should see some data returned.


Conclusion
Congratulations on successfully completing this hands-on lab!

Additional Resources
Your company, LimeDrop, is using Prometheus to monitor their infrastructure. Recently, they have been making use of containers to manage applications. A new Docker server has been created, and the Docker daemon running on that server needs monitoring.
Your task is to set up Prometheus monitoring for this Docker daemon. Configure the Docker daemon to expose metrics, and then configure Prometheus to scrape them.
Additional info:
You can find documentation on exposing Docker metrics in the Docker documentation.
The Docker server is reachable from the Prometheus server at the hostname limedrop-docker.

Docker Container Monitoring with Prometheus
Introduction
Prometheus has exporters that are capable of easily exposing metrics for a variety of systems and infrastructures. Docker containers are a great way to manage your applications, and they can be monitored by Prometheus using cAdvisor. In this lab, you will have the opportunity to use cAdvisor to set up Prometheus monitoring for containers running on a Docker host.
Solution
Log in to the Docker server using the credentials provided:
ssh cloud_user@<DOCKER_SERVER_PUBLIC_IP>
Set Up cAdvisor to Expose Metrics
Run cAdvisor in a container:

 docker run -d --restart always --name cadvisor -p 8080:8080 -v "/:/rootfs:ro" -v "/var/run:/var/run:rw" -v "/sys:/sys:ro" -v "/var/lib/docker/:/var/lib/docker:ro" google/cadvisor:latest


Verify you can query cAdvisor for metrics:

 curl localhost:8080/metrics

 We should see a lot of data returned.


Configure Prometheus to Scrape Docker Container Metrics from cAdvisor
Open a new terminal.


Log in to the Prometheus server:

 ssh cloud_user@<PROMETHEUS_SERVER_PUBLIC_IP>


Edit the Prometheus config:

 sudo vi /etc/prometheus/prometheus.yml


Under the scrape_configs section, add a scrape configuration for cAdvisor:

 - job_name: 'Docker Containers' static_configs: - targets: ['limedrop-docker:8080']


Save and exit the file by pressing Escape followed by :wq.


Restart Prometheus to load the new configuration:

 sudo systemctl restart prometheus


Access the expression browser in a web browser at http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090.


Run a query to view some Docker container metric data:

 container_memory_usage_bytes{name=~"web."}

 We should see some data returned.


Conclusion
Congratulations on successfully completing this hands-on lab!

Additional Resources
Your company, LimeDrop, is using Docker containers to manage applications. They would like to set up Prometheus monitoring for these containers.
Your task is to run and configure cAdvisor to provide Prometheus metrics for your Docker containers. Then configure Prometheus to scrape those metrics.
Additional info:
You can find documentation on exposing Docker container metrics with cAdvisor in the Prometheus documentation.
The Docker server is reachable from the Prometheus server at the hostname limedrop-docker.

Kubernetes Monitoring with Prometheus
Introduction
Prometheus is capable of monitoring a wide variety of environments, including Kubernetes. In this lab, you will have the opportunity to work with both Prometheus and Kubernetes as you configure Prometheus monitoring for a Kubernetes cluster. You will use kube-state-metrics to provide Kubernetes metric data, and you will configure Prometheus to pull in those metrics.
Solution
Log in to the Kubernetes server using the credentials provided:
ssh cloud_user@<KUBERNETES_SERVER_PUBLIC_IP>
Set Up kube-state-metrics to Expose Metrics for Your Kubernetes Cluster
Clone the kube-state-metrics project on GitHub:

 git clone https://github.com/kubernetes/kube-state-metrics.git


Change directory:

 cd kube-state-metrics/


Set it to use the version of kube-state-metrics we want to use:

 git checkout v1.8.0


Apply the YAML files inside the kubernetes directory:

 kubectl apply -f kubernetes


Change back to the home directory:

 cd ..


Create a NodePort service to expose the metrics service outside the cluster, making it accessible to the Prometheus server:

 vi kube-state-metrics-nodeport-svc.yml


Define the NodePort service in a YAML file:

 kind: Service apiVersion: v1 metadata: namespace: kube-system name: kube-state-nodeport spec: selector: k8s-app: kube-state-metrics ports: - protocol: TCP port: 8080 nodePort: 30000 type: NodePort


Save and exit the file by pressing Escape followed by :wq.


Create the NodePort service:

 kubectl apply -f kube-state-metrics-nodeport-svc.yml


Test it:

 curl localhost:30000/metrics

 We should see a lot of data returned.


Configure Prometheus to Scrape Metrics from kube-state-metrics
Open a new terminal.


Log in to the Prometheus server:

 ssh cloud_user@<PROMETHEUS_SERVER_PUBLIC_IP>


Edit the Prometheus config:

 sudo vi /etc/prometheus/prometheus.yml


Under the scrape_configs section, add a scrape configuration for Kubernetes:

 - job_name: 'Kubernetes' static_configs: - targets: ['limedrop-kube:30000']


Save and exit the file by pressing Escape followed by :wq.


Restart Prometheus to load the new configuration:

 sudo systemctl restart prometheus


Access the expression browser in a web browser at http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090.


Run a query to view some Kubernetes metric data:

 kube_pod_status_ready{namespace="default",condition="true"}

 We should see some data returned.


Conclusion
Congratulations on successfully completing this hands-on lab!

Additional Resources
Your company, LimeDrop, is using Kubernetes to orchestrate containers. A Prometheus server is already in place outside the cluster, but the Kubernetes containers are not currently being monitored.
Your task is to use kube-state-metrics to provide metric data for Kubernetes. Then, configure Prometheus to scrape metrics from kube-state-metrics.
Additional info:
You can find documentation on kube-state-metrics on GitHub.
Use kube-state-metrics version 1.8.0.
The Kubernetes server is reachable from the Prometheus server at the hostname limedrop-kube.

Installing Prometheus Pushgateway
Introduction
The Prometheus pull model provides a highly resilient method for gathering metrics. However, there are a few cases where this model is not a good fit. Prometheus Pushgateway serves as a middleman, providing a push-based method for gathering metrics without compromising the simplicity of Prometheus server. In this lab, you will have the opportunity to install and configure your own Prometheus Pushgateway instance. This will give you some hands-on experience with the process of setting up Pushgateway.
Solution
Log in to the Prometheus server using the credentials provided:
ssh cloud_user@<PROMETHEUS_SERVER_PUBLIC_IP>
Install and Run Pushgateway
Create a user and group for Pushgateway:

 sudo useradd -M -r -s /bin/false pushgateway


Download the archive that contains the Pushgateway binary:

 wget https://github.com/prometheus/pushgateway/releases/download/v1.2.0/pushgateway-1.2.0.linux-amd64.tar.gz


Extract the file:

 tar xvfz pushgateway-1.2.0.linux-amd64.tar.gz


Copy it to /usr/local/bin/:

 sudo cp pushgateway-1.2.0.linux-amd64/pushgateway /usr/local/bin/


Set ownership on the file:

 sudo chown pushgateway:pushgateway /usr/local/bin/pushgateway


Create a systemd unit file for Pushgateway:

 sudo vi /etc/systemd/system/pushgateway.service


Add the following to the file:

 [Unit] Description=Prometheus Pushgateway Wants=network-online.target After=network-online.target [Service] User=pushgateway Group=pushgateway Type=simple ExecStart=/usr/local/bin/pushgateway [Install] WantedBy=multi-user.target


Save and exit the file by pressing Escape followed by :wq.


Start and enable the pushgateway service:

 sudo systemctl enable pushgateway

 sudo systemctl start pushgateway


Verify the service is running:

 sudo systemctl status pushgateway

 We should see it as an active (running) status.


Verify it's serving metrics:

 curl localhost:9091/metrics

 We should see data returned.


Configure Pushgateway as a Scrape Target for Prometheus Server
Edit the Prometheus config:

 sudo vi /etc/prometheus/prometheus.yml


Under the scrape_configs section, add a scrape configuration for Pushgateway. Be sure to set honor_labels: true:

 - job_name: 'Pushgateway' honor_labels: true static_configs: - targets: ['localhost:9091']


Restart Prometheus to load the new configuration:

 sudo systemctl restart prometheus


Access the expression browser in a web browser at http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090.


Run a query to view some Pushgateway metric data:

 pushgateway_build_info

 We should see some data returned.


Conclusion
Congratulations on successfully completing this hands-on lab!

Additional Resources
Your company, LimeDrop, is using Prometheus for monitoring. However, they have some short-lived batch jobs they would like to monitor. Unfortunately, the pull model used by Prometheus server to scrape metric data will not work for these jobs, since they terminate when processing is finished.
LimeDrop needs a Prometheus Pushgateway instance to which these jobs can push metric data. This will allow those metrics to be scraped by Prometheus server. Install Pushgateway on the existing Prometheus server and configure Prometheus server to scrape from Pushgateway.

Monitoring a Batch Job with Prometheus Pushgateway
Introduction
Prometheus Pushgateway provides a way to provide metrics to Prometheus with a push-based model. This is particularly useful for monitoring short-lived job processes. In this lab, you will have the opportunity to work with the Pushgateway API by pushing metrics to it. You will modify a simple job to implement monitoring for the job by pushing metrics to Pushgateway every time it runs.
Solution
Log in to the Job server using the credentials provided:
ssh cloud_user@<JOB_SERVER_PUBLIC_IP>
Modify the Cleanup Job to Push a Metric to Pushgateway to Signal When It Runs
Open the cleanup job script:

 sudo vi /etc/jobs/cleanup.sh


Edit the file to implement a call to the Pushgateway API at the end of the script to signal that the job ran:

 num_files=$(rm -vrif /etc/debug_data/* | wc -l) cat << EOF | curl --data-binary @- http://prometheus:9091/metrics/job/debug_cleanup/instance/10.0.1.102 # TYPE job_executed_successful gauge job_executed_successful 1 EOF


Save the file by pressing Escape followed by :w.


Access the Prometheus expression browser: http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090/graph.


Run a query to verify you can see metric data pushed to Pushgateway by the cleanup job script (you may have to wait a minute or so for the job to execute so you can see your changes):

 job_executed_successful[5m]

 We should see some data returned.


Modify the Cleanup Job to Push a Metric to Pushgateway Representing the Number of Files Deleted in Each Execution
Edit the cleanup job script to implement a call to the Pushgateway API at the end of the script to signal that the job ran:

 num_files=$(rm -vrif /etc/debug_data/* | wc -l) cat << EOF | curl --data-binary @- http://prometheus:9091/metrics/job/debug_cleanup/instance/10.0.1.102 # TYPE job_executed_successful gauge job_executed_successful 1 # TYPE job_num_files_deleted gauge job_num_files_deleted $num_files EOF


Save and exit the file by pressing Escape followed by :wq.


In the expression browser, verify you can see the new metric in Prometheus (note you may have to wait a minute or so for the job to execute so you can see your changes):

 job_num_files_deleted[5m]

 We should see some data returned.


Conclusion
Congratulations on successfully completing this hands-on lab!

Additional Resources
Your company, LimeDrop, is using Prometheus to monitor their infrastructure. There is a simple Bash cleanup script that removes unneeded files periodically, but this job is not monitored. Since this job is a short-lived process, it can use Pushgateway to send metrics to Prometheus using a push model.
Implement monitoring for the job by modifying the cleanup script to send some metrics to Pushgateway.
Some additional details:
The cleanup script is located on the Job server at /etc/jobs/cleanup.sh. You can implement monitoring by making changes to this script.
The script is already set up to run approximately once every minute.
You can reach Pushgateway from the Job server at prometheus:9091.
Implement a simple gauge metric for the job called job_executed_successful. Push this metric with a value of 1 every time the script completes execution.
Implement a gauge metric for the job called job_num_files_deleted. Every time the job executes, send push metric with a value representing the number of files cleaned up by the script. The script already contains a variable called num_files you can use to get the number of files.

Using Prometheus Recording Rules
Introduction
Prometheus is a great way to collect a variety of metrics for your servers and applications. However, sometimes you need to perform calculations on the raw data Prometheus collects in order to gain actionable information. While queries can help you do this, recording rules provide an additional layer of optimization by allowing you to periodically pre-calculate query results and save them. In this lab, you will have the opportunity to implement some basic recording rules. This will provide you with some hands-on familiarity with the process of creating and managing recording rules.
Solution
Log in to the Prometheus server using the credentials provided:
ssh cloud_user@<PROMETHEUS_SERVER_PUBLIC_IP>
Implement a Recording Rule to Pre-Calculate CPU Usage
Create a directory to store rules files:

 sudo mkdir -p /etc/prometheus/rules


Open the Prometheus config:

 sudo vi /etc/prometheus/prometheus.yml


Edit it to add a rules location:

 ... rule_files: - "/etc/prometheus/rules/*.yml" ...


Save and exit the file by pressing Escape followed by :wq.


Create a rules file for limedrop-gateway server metrics:

 sudo vi /etc/prometheus/rules/limedrop-gateway.yml


Implement a recording rule to pre-calculate CPU usage for the server:

 groups: - name: limedrop_gateway rules: - record: limedrop_gateway:cpu_usage expr: sum(rate(node_cpu_seconds_total{instance='limedrop-gateway:9100',mode!='idle'}[5m])) * 100 / 2


Save and exit the file by pressing Escape followed by :wq.


Restart Prometheus to load the new configuration:

 sudo systemctl restart prometheus


Access the expression browser: http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090/graph.


Run a query to verify you can see the pre-calculated data:

 limedrop_gateway:cpu_usage[5m]


Add a Recording Rule to Pre-Calculate Memory Usage
On the Prometheus server, open the rules file:

 sudo vi /etc/prometheus/rules/limedrop-gateway.yml


Edit it to add a recording rule to pre-calculate memory usage for the server:

 groups: - name: limedrop_gateway rules: - record: limedrop_gateway:cpu_usage expr: sum(rate(node_cpu_seconds_total{instance='limedrop-gateway:9100',mode!='idle'}[5m])) * 100 / 2 - record: limedrop_gateway:memory_usage expr: 100 - (node_memory_MemAvailable_bytes{instance='limedrop-gateway:9100'} / node_memory_MemTotal_bytes{instance='limedrop-gateway:9100'}) * 100


Save and exit the file by pressing Escape followed by :wq.


Restart Prometheus to load the new configuration:

 sudo systemctl restart prometheus


Access the expression browser: http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090/graph.


Run a query to verify you can see the pre-calculated data:

 limedrop_gateway:memory_usage[5m]


You can also check out the rules interface (http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090/rules) to see the status of your rules.


Conclusion
Congratulations on successfully completing this hands-on lab!


Additional Resources
Your company, LimeDrop, is using Prometheus to monitor several servers. One such server, limedrop-gateway, is running an API gateway application. The admin team has a dashboard set up to view performance statistics for this server.
This means the queries calculating these statistics are executed every time the dashboard refreshes. You have been asked to optimize your usage of Prometheus resources by building some recording rules to pre-calculate some of the performance statistics for the limedrop-gateway server.
Implement the requested recording rules and query their data to make sure they are working.
Here are the details:
The instance name for the server is limedrop-gateway:9100.


Build a recording rule to pre-calculate CPU usage data for the server. Store the results in a metric called limedrop_gateway:cpu_usage. You can use this query to get CPU usage:

 sum(rate(node_cpu_seconds_total{instance='limedrop-gateway:9100',mode!='idle'}[5m])) * 100 / 2


Build a recording rule to pre-calculate memory usage data for the server. Store the results in a metric called limedrop_gateway:memory_usage You can use this query to get memory usage:

 100 - (node_memory_MemAvailable_bytes{instance='limedrop-gateway:9100'} / node_memory_MemTotal_bytes{instance='limedrop-gateway:9100'}) * 100



Installing Prometheus Alertmanager
Introduction
Prometheus alerts allow you to issue automated notifications when certain events occur, triggered by your metric data. Alertmanager is a necessary component in this process, handling the process of sending alerts to the appropriate destination, as well as adding some additional control over alerting functionality. In this lab, you will have the opportunity to install and configure an Alertmanager instance, and connect it to an existing Prometheus server.
Solution
Log in to the Prometheus server using the credentials provided:
ssh cloud_user@<PROMETHEUS_SERVER_PUBLIC_IP>
Install Alertmanager
Create a user and group for Alertmanager:

 sudo useradd -M -r -s /bin/false alertmanager


Download the Alertmanager binary:

 wget https://github.com/prometheus/alertmanager/releases/download/v0.20.0/alertmanager-0.20.0.linux-amd64.tar.gz


Extract the file:

 tar xvfz alertmanager-0.20.0.linux-amd64.tar.gz


Copy the Alertmanager binary file to /usr/local/bin/:

 sudo cp alertmanager-0.20.0.linux-amd64/alertmanager /usr/local/bin/


Set the ownership:

 sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager


Make the directory that will contain the configuration files:

 sudo mkdir -p /etc/alertmanager


Move the default configuration file to the new directory:

 sudo cp alertmanager-0.20.0.linux-amd64/alertmanager.yml /etc/alertmanager


Set the ownership:

 sudo chown -R alertmanager:alertmanager /etc/alertmanager


Create a directory to serve as local storage for Alertmanager to use:

 sudo mkdir -p /var/lib/alertmanager


Set the ownership:

 sudo chown alertmanager:alertmanager /var/lib/alertmanager


Create a systemd unit file:

 sudo vi /etc/systemd/system/alertmanager.service


Add the following to the file to download and install the Alertmanager binaries:

 [Unit] Description=Prometheus Alertmanager Wants=network-online.target After=network-online.target [Service] User=alertmanager Group=alertmanager Type=simple ExecStart=/usr/local/bin/alertmanager \ --config.file /etc/alertmanager/alertmanager.yml \ --storage.path /var/lib/alertmanager/ [Install] WantedBy=multi-user.target


Save and exit the file by pressing Escape followed by :wq.


Enable the alertmanager service so it starts automatically on boot:

 sudo systemctl enable alertmanager


Start the alertmanager service:

 sudo systemctl start alertmanager


Verify the service is running:

 sudo systemctl status alertmanager


Verify you can reach it:

 curl localhost:9093


You can also access Alertmanager in a web browser at http://<PROMETHEUS_SERVER_PUBLIC_IP>:9093.


Install amtool
Install the amtool binary:

 sudo cp alertmanager-0.20.0.linux-amd64/amtool /usr/local/bin/


Make the directory that will contain the configuration file:

 sudo mkdir -p /etc/amtool


Create a config file for amtool:

 sudo vi /etc/amtool/config.yml


Enter the following content in the config file:

 alertmanager.url: http://localhost:9093


Save and exit the file by pressing Escape followed by :wq.


Verify amtool is working by pulling the current Alertmanager configuration:

 amtool config show

 We should see the default configuration.


Configure Prometheus to Use Alertmanager
Open the Prometheus config file:

 sudo vi /etc/prometheus/prometheus.yml


Under alerting, add your Alertmanager as a target:

 alerting: alertmanagers: - static_configs: - targets: ["localhost:9093"]


Save and exit the file by pressing Escape followed by :wq.


Restart Prometheus to reload the configuration:

 sudo systemctl restart prometheus


Access the Prometheus Expression Browser in a web browser at http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090/graph.


Enter the following in the query box:

 prometheus_notifications_alertmanagers_discovered


Click Execute.


Ensure the current value is 1.


Conclusion
Congratulations on successfully completing this hands-on lab!

Additional Resources
Your company, LimeDrop, is using Prometheus for monitoring. There have been some recurring issues with a few critical services, and they would like to set up alerts so the team can act quickly when something goes wrong. In order to do this, they will need Prometheus Alertmanager.
Your task is to install and configure Alertmanager and amtool, then configure Prometheus to use Alertmanager. You can install Alertmanager directly on the provided Prometheus server, which already has Prometheus installed and running.

Configuring Prometheus Alertmanager
Introduction
Prometheus Alertmanager provides useful functionality around processing and managing alerts triggered by your Prometheus metric data. In this lab, you will have the opportunity to gain some hands-on experience with the process of configuring Alertmanager itself. You will be able to work with a real Alertmanager instance, making configuration changes to it.
Solution
Log in to the Prometheus server using the credentials provided:
ssh cloud_user@<PROMETHEUS_SERVER_PUBLIC_IP>
Set the from Address for Email Alerts
Edit the Alertmanager configuration file:

 sudo vi /etc/alertmanager/alertmanager.yml


Set global.smtp_from to noreply@limedrop.com:

 global: smtp_from: noreply@limedrop.com ...


Add the Requested Templates Directory
Beneath the global block, add the requested directory as a notification template location:

 templates: - "/etc/alertmanager/templates/*.tmpl"


Save and exit the file by pressing Escape followed by :wq.


Restart Alertmanager to load the new configuration:

 sudo systemctl restart alertmanager


Verify your configuration is valid by ensuring Alertmanager is running. Access Alertmanager in your browser: http://<PROMETHEUS_SERVER_PUBLIC_IP>:9093


Once you can see Alertmanager in your browser, click Status and look for your new configuration options in the Config section.


Conclusion
Congratulations on successfully completing this hands-on lab!

Additional Resources
Your company, LimeDrop, has recently set up an Alertmanager instance to handle alerts from a Prometheus server. However, after using Alertmanager for a week, it has become apparent that a few configuration tweaks are needed. Implement the requested configuration changes in the existing Alertmanager instance running on the Prometheus server.
If you wish, you can read more about the configuration options available for Alertmanager in the Prometheus documentation.
Implement the following configuration changes:
The company would like email alerts to have a from address of noreply@limedrop.com to ensure users know they should not directly reply to alert emails.
Alertmanager has the ability to load notification templates from files. The company would like to give some employees the ability to add new templates without giving them access to the entire Prometheus server. As such, a special directory has been created with appropriate permissions in order to house these templates. Configure Alertmanager to read templates from the directory located at /etc/alertmanager/templates/.

Configuring Prometheus Alertmanager for High Availability
Introduction
Prometheus Alertmanager is a great way to handle your Prometheus alerts. However, a lone instance of Alertmanager can serve as a single point of failure if it goes down. Luckily, you can configure Alertmanager to run in a multi-instance cluster to provide failure resilience. In this hands-on lab, you will make an existing single-instance Alertmanager setup highly available by adding an additional instance.
Solution
Log in to the Prometheus Server using the credentials provided:
ssh cloud_user@<PROMETHEUS_SERVER_PUBLIC_IP>
Log in to the Alertmanager 2 server using the credentials provided:
ssh cloud_user@<ALERTMANAGER_2_PUBLIC_IP>
Configure the Two Alertmanager Instances to Form a Cluster
On both servers, edit the Alertmanager unit file:

 sudo vi /etc/systemd/system/alertmanager.service


In the ExecStart section of the Prometheus Server, add the private IP address of the Alertmanager 2 server using the cluster.peer flag:

 ExecStart=/usr/local/bin/alertmanager \ --config.file /etc/alertmanager/alertmanager.yml \ --storage.path /var/lib/alertmanager/ \ --cluster.peer=10.0.1.102:9094


 Note: When you add the cluster.peer line, be sure to add the \ at the end of the storage.path line.



Save and exit the file by pressing Escape followed by :wq.


In the ExecStart section of the Alertmanager 2 server, add the private IP address of the Prometheus Server using the cluster.peer flag:

 ExecStart=/usr/local/bin/alertmanager \ --config.file /etc/alertmanager/alertmanager.yml \ --storage.path /var/lib/alertmanager/ \ --cluster.peer=10.0.1.101:9094


 Note: When you add the cluster.peer line, be sure to add the \ at the end of the storage.path line.



Save and exit the file by pressing Escape followed by :wq.


On both servers, reload the unit file:

 sudo systemctl daemon-reload


On the Prometheus Server, restart Alertmanager:

 [cloud_user@prometheus_server]$ sudo systemctl restart alertmanager


On the Alertmanager 2 server, enable and start Alertmanager:

 [cloud_user@alertmanager_2]$ sudo systemctl enable alertmanager

 [cloud_user@alertmanager_2]$ sudo systemctl start alertmanager


On both servers, check the status of Alertmanager:

 sudo systemctl status alertmanager

 They should both show a status of active (running).


On both servers, press Ctrl+C to exit the process.


Access the Prometheus Server instance in a new browser tab: http://<PROMETHEUS_SERVER_PUBLIC_IP>:9093


Click Silences.


Click New Silence.


Set the following values:


Name: test
Value: 1
Creator: me
Comment: This is a test
Click Create.


Access the Alertmanager 2 instance in a new browser tab: http://<ALERTMANAGER_2_PUBLIC_IP>:9093


Click Silences, and verify the silence you created on the Prometheus Server instance appears.


Click View, and we should then see it's the one we created.


Configure Prometheus to Use Your Multi-Instance Alertmanager Setup
Close out of the Alertmanager 2 terminal, as we will no longer be working in it.


On the Prometheus Server, edit the Prometheus configuration file:

 [cloud_user@prometheus_server]$ sudo vi /etc/prometheus/prometheus.yml


Add the new Alertmanager (10.0.1.102:9093) to the list of Alertmanager targets:

 alerting: alertmanagers: - static_configs: - targets: - localhost:9093 - 10.0.1.102:9093


Save and exit the file by pressing Escape followed by :wq.


Restart Prometheus to reload the config:

 [cloud_user@prometheus_server]$ sudo systemctl restart prometheus


Access the Prometheus server in the browser: http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090


Click Status > Runtime & Build Information.


Verify both of your Alertmanagers appear under the Alertmanagers section.


Conclusion
Congratulations on successfully completing this hands-on lab!
Additional Resources
Your company, LimeDrop, is using Alertmanager to handle Prometheus alerts. Recently, there was an outage in part of the data center. The single Alertmanager instance was one of the affected servers, and the problem was not detected for a few hours because no one received an alert. Your task is to build a multi-instance configuration by adding an additional Alertmanager instance.
The Prometheus server has the first Alertmanager instance running on it. A server called Alertmanager 2 has already been set up. Alertmanager 2 will run the second Alertmanager instance. Alertmanager is installed there, but is not running.
Configure both Alertmanager instances to run as a two-instance cluster. Then, configure Prometheus server so it is able to send alerts to both instances as needed.

Configuring Prometheus Alerts
Introduction
Prometheus alerts allow you to issue real-time notifications triggered by your Prometheus metric data. In this lab, you will work directly with Prometheus alerts. You will create an alert in an existing Prometheus server by modifying the Prometheus configuration.
Solution
Log in to the Prometheus server using the credentials provided:
ssh cloud_user@<PROMETHEUS_SERVER_PUBLIC_IP>
Create an Alerting Rule in Prometheus to Alert When the Server Goes Down
Open the Prometheus config file:

 sudo vi /etc/prometheus/prometheus.yml


Add a path for rules files to the Prometheus config:

 rule_files: - "/etc/prometheus/rules/*.yml"


Save and exit the file by pressing Escape followed by :wq.


Create the rules directory:

 sudo mkdir -p /etc/prometheus/rules/


Create a new rules file for your alerting rule:

 sudo vi /etc/prometheus/rules/limedrop-web.yml


Add the following content to the file to implement an alerting rule to issue an alert when the server goes down:

 groups: - name: limedrop-web rules: - alert: WebServerDown expr: up{instance="limedrop-web:9100"} == 0 labels: severity: critical annotations: summary: Web Server Down


Save and exit the file by pressing Escape followed by :wq.


Restart Prometheus to reload the configuration:

 sudo systemctl restart prometheus


Access Prometheus in a browser at http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090.


Click Alerts. You should see your WebServerDown alert listed.


Configure a Routing Tree and Receiver in Alertmanager
In the terminal, edit the Alertmanager configuration file:

 sudo vi /etc/alertmanager/alertmanager.yml


Add a new node to the routing table:

 route: ... routes: - receiver: 'web.hook' group_wait: 30s match_re: alertname: WebServerDown


Save and exit the file by pressing Escape followed by :wq.


Restart Alertmanager to reload the configuration:

 sudo systemctl restart alertmanager


You can test your alert by shutting down the limedrop-web exporter to simulate the server going down. To do this, stop the limedrop_web_exporter service:

 sudo systemctl stop limedrop_web_exporter


Refresh the Prometheus alerts page, where we should see the WebServerDown alert is now active.


In a new browser tab, navigate to the Alertmanager page tab: http://<PROMETHEUS_SERVER_PUBLIC_IP>:9093. If the alert is firing, you should see the WebServerDown alert appear.


In the terminal, start limedrop_web_exporter again:

 sudo systemctl start limedrop_web_exporter


After a minute or so, refresh the Prometheus alerts page. We should see the alert turn green.


Refresh the Alertmanager page, where we should see the alert has disappeared.


Conclusion
Congratulations on successfully completing this hands-on lab!
Additional Resources
Your company, LimeDrop, is using Prometheus to monitor some systems and applications. They have a web server called limedrop-web that is having some stability issues. They would like to issue an alert to Alertmanager whenever this server goes down. Configure Prometheus and Alertmanager to handle this alert.
You will need to:
Configure an alerting rules directory in Prometheus.
Add an alerting rule to fire an alert when the server goes down. You can use the expression up{instance="limedrop-web:9100"} == 0.
Add a routing tree node to the Alertmanager config.
You can simulate the server going down to fire the alert by stopping its exporter:
sudo systemctl stop limedrop_web_exporter


Advanced Configuration for Prometheus Alerts
Introduction
Prometheus Alertmanager provides some additional useful features around the management of alerts. These features allow you to customize and tweak your alerts so they are more useful in real-world situations. In this lab, you will have the opportunity to practice using some of these Alertmanager features, including alert grouping, inhibitions, and silences.
Solution
Log in to the Prometheus server using the credentials provided:
ssh cloud_user@<PROMETHEUS_SERVER_PUBLIC_IP>
Combine the Web Server Down Alerts into a Single Group
Check Prometheus in a web browser at http://<PROMETHEUS_SERVER_PUBLIC_IP>:9090.


Click the Alerts tab. We should see the WebBadGateway alert as well as WebServer1Down and WebServer2Down.


In the terminal, open the Alertmanager configuration file:

 sudo vi /etc/alertmanager/alertmanager.yml


Add a new node to routing tree to combine the WebServer.*Down alerts:

 route: ... routes: - receiver: 'web.hook' group_by: ['service'] match_re: alertname: 'WebServer.*Down'


Save and exit the file by pressing Escape followed by :wq.


Load the new configuration:

 sudo killall -HUP alertmanager


Check Alertmanager in a web browser at http://<PROMETHEUS_SERVER_PUBLIC_IP>:9093. You should see the Web Server alerts grouped together under the group service="webserver".


Create an Inhibition to Stop the WebBadGateway Alert When a WebServerDown Alert Is Already Firing
In the terminal, edit the Alertmanager configuration file:

 sudo vi /etc/alertmanager/alertmanager.yml


Add a new inhibit rule:

 inhibit_rules: ... - source_match_re: alertname: 'WebServer.*Down' target_match: alertname: 'WebBadGateway'


Save and exit the file by pressing Escape followed by :wq.


Load the new configuration:

 sudo killall -HUP alertmanager


Refresh Alertmanager in the browser. The WebBadGateway should no longer appear. You can click the Inhibited box to make it appear again.


Silence the WebServer1Down Alert
Expand the service="webserver" group.


Locate the alert with alertname="WebServer1Down", and click the Silence button for that alert.


Enter your name for Creator.


Enter "silence WebServerDown" for Comment.


Click Create.


Navigate back to the main Alertmanager page, where we should see WebServer1Down is no longer there.


Conclusion
Congratulations on successfully completing this hands-on lab!
Additional Resources
Your company, LimeDrop, is using Alertmanager to handle Prometheus alerts. Alertmanager is set up to issue alerts when there are problems with the company's main website. The website is currently experiencing issues. Since you are the expert on Prometheus, the admin team has asked you to perform some tweaks in Alertmanager to make the alerts more relevant and useful.
Implement the following changes in Alertmanager:
There is a collection of web servers, and when issues arise there is usually more than one instance that is down at the same time. However, when multiple instances go down, the team gets a separate alert for each instance. Ensure these alerts are combined into a group in Alertmanager so there is only one alert message even if multiple web servers are down. The relevant alerts all have names that match the expression WebServer.*Down.
When web servers go down, the website will begin to respond with 502 (Bad Gateway) error messages. This message triggers an additional alert. However, when there are already alerts about the servers being down, this additional alert is unnecessary and distracting. Configure Alertmanager to inhibit the alert named WebBadGateway whenever any of the WebServer.*Down alerts are firing.
Web Server 1 is repeatedly going down and then recovering, resulting in multiple alert notifications. Temporarily silence the WebServer1Down alert for two hours.

Building a Highly Available Prometheus Cluster
In this lab, we are setting up a second Prometheus instance to support our first. This way, if one goes down, it will be more likely that we can still obtain metric data.
Prometheus Server 1 is an existing server that is already running. Prometheus Server 2 has Prometheus installed, but it is not configured or running. We will configure Prometheus Server 2 to serve as an additional instance alongside Prometheus Server 1 so that monitoring services are more highly available.
Before We Begin
Using two separate terminals, log in to both of the Prometheus servers using the provided credentials.
Copy the Prometheus Configuration
Copy the Prometheus configuration from the existing Prometheus server to the new instance by doing the following:
Make sure we're in Prometheus Server 1.


Copy prometheus.yml to Prometheus Server 2:

 scp /etc/prometheus/prometheus.yml cloud_user@10.0.1.102:/home/cloud_user

 Use y when prompted and use the provided password for Prometheus Server 2.


Log in to Prometheus Server 2.


Copy prometheus.yml to the appropriate location:

 sudo cp ~/prometheus.yml /etc/prometheus/prometheus.yml


Configure the Rules Configuration
Copy the rules configuration from the existing Prometheus server to the new instance.
On Prometheus Server 1, see what information is in our rules directory:

 ls /etc/prometheus/rules/

 Note the limedrop-alerts.yml as this is the file we are going to transfer.


Copy files from the rules directory to Prometheus Server 2:

 scp /etc/prometheus/rules/* cloud_user@10.0.1.102:/home/cloud_user


On Prometheus Server 2, create the rules directory:

 sudo mkdir -p /etc/prometheus/rules


Copy the rules file to the appropriate locations:

 sudo cp ~/limedrop-alerts.yml /etc/prometheus/rules


Start and Verify the new Prometheus Instance
Start the new Prometheus instance and verify that everything is working.
On Prometheus Server 2, enable Prometheus.

 sudo systemctl enable prometheus


Start Prometheus:

 sudo systemctl start prometheus


Access Prometheus Server 2 in a browser at http://<Prometheus Server 2 Public IP>:9090.


Run a query to verify that it is scraping metrics from limedrop-web.

 up{instance="limedrop-web:9100"}


You can also click Alerts to verify that the WebServerDown alert appears.


Conclusion
Congratulations! You've completed the lab!

Additional Resources
Your company, LimeDrop, is using Prometheus to monitor a variety of applications and servers. Recently, a major outage occurred which caused the Prometheus server itself to go down, severely impacting the team's ability to discover what went wrong since they could not access any metric data.
The company would like to ensure that Prometheus is more highly available by setting up a second Prometheus instance. This way, if one goes down, it will be more likely that the team can still obtain metric data.
Prometheus Server 1 is an existing server that is already running. Prometheus Server 2 has Prometheus installed, but it is not configured or running. Your task is to configure Prometheus Server 2 to serve as an additional instance alongside Prometheus Server 1 so that monitoring services are more highly available.


Implementing Hierarchical Federation with Prometheus
Introduction
Prometheus servers are capable of monitoring a large number of applications and components. However, it does not always make sense to monitor everything with a single Prometheus server. Prometheus provides the ability to federate Prometheus servers, allowing high-level Prometheus servers to collect, and even aggregate, metric data from multiple low-level Prometheus servers. In this lab, you will be able to see how this works. You will configure a high-level Prometheus instance to collect data from multiple lower-level Prometheus servers in a single location.
Solution
Log in to the Federal Prometheus Server via SSH using the provided credentials.
Configure the Federal Prometheus Server to Scrape Metrics from the Other Two Prometheus Servers
Edit the Prometheus configuration:

 sudo vi /etc/prometheus/prometheus.yml


Implement a scrape configuration to pull metrics from the other two Prometheus servers. Under this section:

 scrape_configs: -targets:['localhost:9090']

 Enter the following:

 ... - job_name: 'federate' scrape_interval: 15s honor_labels: true metrics_path: '/federate' params: 'match[]': - '{job!~"prometheus"}' static_configs: - targets: - 'prometheus-ds-1:9090' - 'prometheus-ds-2:9090'


Save and quit by pressing Escape followed by :wq.


Start the Federal Prometheus Server and Verify Everything Is Working
Enable Prometheus on the Federal Prometheus Server:

 sudo systemctl enable prometheus


Start Prometheus:

 sudo systemctl start prometheus


Access the Federal Prometheus Server in a browser at http://<Federated Prometheus Server Public IP>:9090. (The public IP can be found in the lab credentials.)


Run a query to view some metric data:

 up

 We should see metric data for instances that are monitored by the other two Prometheus servers. These instances are called limedrop-web-1:9100 and limedrop-web-2:9100.


Conclusion
Congratulations! You've completed the lab!

Additional Resources
Your company, LimeDrop, has two data centers running their infrastructure. Each data center has its own Prometheus server, responsible for collecting metric data from application and components running in that data center. However, they would like to be able to collect all this data in one place so they can view metrics for the entire infrastructure across both data centers.
Your task is to set up a third, federated Prometheus server to gather metrics from the two existing data center-specific Prometheus servers. Federal Prometheus Server has been set up for this purpose. Prometheus is already installed on that server, but it is not configured or running. Configure Federal Prometheus Server to federate metric data from the other two data center-specific Prometheus servers, and then get Federal Prometheus Server up and running.
Note: You can reach the other two Prometheus servers from the Federal Prometheus Server using either private IP addresses or the hostnames prometheus-ds-1 and prometheus-ds-2.

Using the Java Client Library for Prometheus
In this lab, we will use the Prometheus Java client libraries to collect metrics for the /limesAvailable endpoint and expose a metrics endpoint that Prometheus can scrape. Then, test it by configuring Prometheus to scrape metrics from the app.
Add instrumentation to track the following metrics for the /limesAvailable endpoint:
total_requests (counter) — The total number of requests processed by the /limesAvailable endpoint during the lifetime of the application process.
inprogress_requests (gauge) — The number of requests currently being processed by the limesAvailable endpoint.
Some additional information you will need to be aware of:
The Java source code can be found on the Prometheus server at /home/cloud_user/content-prometheusdd-limedrop-svc.
From the project directory, you can run the application with the command ./gradlew clean bootRun.
While the application is running, you can access it using port 8080 on the Prometheus server.
Inside the Java project, the logic for the /limesAvailable endpoint can be found in src/main/java/com/limedrop/svc/LimeDropController.java.
The main Application class is src/main/java/com/limedrop/svc/App.java.
You can also find the application source code in GitHub at https://github.com/linuxacademy/content-prometheusdd-limedrop-svc. Check the example-solution branch for an example of the code changes needed to complete this lab.
Before We Begin
To get started, we need to log in to the Prometheus Server using the provided credentials.
Add Prometheus Client
Add Prometheus client library dependencies to the Java project.
Change to the root directory for the Java project:

 cd /home/cloud_user/content-prometheusdd-limedrop-svc


Run the project to verify that it is able to compile and run before making any changes:

 ./gradlew clean bootRun


Once we see the text Started App in X seconds, the application is running. Use control + C to stop it.


Edit build.gradle:

 vi build.gradle


Locate the dependencies block and add the following Prometheus client library dependencies:

 dependencies { implementation 'io.prometheus:simpleclient:0.8.1' implementation 'io.prometheus:simpleclient_httpserver:0.8.1' ... }


Run :wq to save and close the file.


If you want, you can run the project again to automatically download the dependencies and make sure it still works: ./gradlew clean bootRun
Add Instrumentation
Add instrumentation to the /limesAvailable endpoint.
Edit the controller class:
 vi src/main/java/com/limedrop/svc/LimeDropController.java


Add the requested counter and gauge metrics so that the file matches the following:
 package com.limedrop.svc; import org.springframework.web.bind.annotation.GetMapping; import org.springframework.web.bind.annotation.RequestMapping; import org.springframework.web.bind.annotation.RestController; import io.prometheus.client.Counter; import io.prometheus.client.Gauge; @RestController public class LimeDropController { static final Counter totalRequests = Counter.build() .name("total_requests").help("Total requests.").register(); static final Gauge inprogressRequests = Gauge.build() .name("inprogress_requests").help("Inprogress requests.").register(); @GetMapping(path="/limesAvailable", produces = "application/json") public String checkAvailability() { inprogressRequests.inc(); totalRequests.inc(); String response = "{\"warehouse_1\": \"58534\", \"warehouse_2\": \"72399\"}"; inprogressRequests.dec(); return response; } }


Run the application again to make sure it compiles.
 ./gradlew clean bootRun


Add a Scrape Endpoint
Add a scrape endpoint to the application:
Edit the main application class.

 vi src/main/java/com/limedrop/svc/App.java


Use the Prometheus HTTPServer to set up a scrape endpoint on port 8081 by editing the file to match the following:

 package com.limedrop.svc; import org.springframework.boot.SpringApplication; import org.springframework.boot.autoconfigure.SpringBootApplication; import io.prometheus.client.exporter.HTTPServer; import java.io.IOException; @SpringBootApplication public class App { public static void main(String[] args) { SpringApplication.run(App.class, args); try { HTTPServer server = new HTTPServer(8081); } catch (IOException e) { e.printStackTrace(); } } }


Run the application again to make sure it compiles:

 ./gradlew clean bootRun


While the application is still running, you should be able to access the /limesAvailable endpoint at http://<Prometheus Server Public IP>:8080. You can view the metrics at http://<Prometheus Server Public IP>:8081/metrics.


Access the /limesAvailable endpoint and see the total_requests counter increase each time you access the endpoint.


Test Your Setup
Test your setup by configuring Prometheus to scrape from your application.
Edit the Prometheus config:
sudo vi /etc/prometheus/prometheus.yml
Add a scrape config to scrape metrics for your app:
scrape_configs: ... - job_name: 'LimeDrop Java Svc' static_configs: - targets: ['localhost:8081']
Restart Prometheus to reload the config:
sudo systemctl restart prometheus
Run your Java app and leave it running to allow Prometheus to collect metrics:
./gradlew clean bootRun
Access Prometheus in a browser at http://<Prometheus Server Public IP>:9090. Run queries to see the metrics you are collecting for your Java app:
total_requests inprogress_requests
Conclusion
Congratulations! You've completed the lab!


Additional Resources
Your company, LimeDrop, is building a RESTful web service using Java. They would like to implement Prometheus monitoring for this application. Since you are the Prometheus expert, the developers have asked you to implement some basic Prometheus monitoring in the Java code. Currently, the application has only one endpoint called /limesAvailable. You will need to implement monitoring around the usage of this endpoint.
Use the Prometheus Java client libraries to collect metrics for the /limesAvailable endpoint and expose a metrics endpoint that Prometheus can scrape. Then, test it by configuring Prometheus to scrape metrics from the app.
Add instrumentation to track the following metrics for the /limesAvailable endpoint:
total_requests (counter) — The total number of requests processed by the /limesAvailable endpoint during the lifetime of the application process.
inprogress_requests (gauge) — The number of requests currently being processed by the limesAvailable endpoint.
Some additional information you will need to be aware of:
The Java source code can be found on the Prometheus server at /home/cloud_user/content-prometheusdd-limedrop-svc.
From the project directory, you can run the application with the command ./gradlew clean bootRun.
While the application is running, you can access it using port 8080 on the Prometheus server.
Inside the Java project, the logic for the /limesAvailable endpoint can be found in src/main/java/com/limedrop/svc/LimeDropController.java.
The main Application class is src/main/java/com/limedrop/svc/App.java.
You can also find the application source code in GitHub at https://github.com/linuxacademy/content-prometheusdd-limedrop-svc. Check the example-solution branch for an example of the code changes needed to complete this lab.

