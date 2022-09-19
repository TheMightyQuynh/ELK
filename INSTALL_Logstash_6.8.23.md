# Setting up Logstash 6.8.23 on AWS
## Prerequisites
3-node cluster running Elasticsearch and Kibana 6.8.23 on AWS. See [Setting up Elasticsearch 6.8.23 on AWS](https://github.com/TheMightyQuynh/ELK/blob/main/1_INSTALL_Elasticsearch_6.8.23.md).

## Install Logstash
1. SSH into the EC2 instance hosting the master-1 node
2. Download and install the Public Signing Key if not already there
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
2. Add the 6.x source list to the `source.list.d` directory where APT can search for new sources
```
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
```
3. Update the package list and install Logstash
```
sudo apt-get update && sudo apt-get install logstash
```
Note: Got the below error when attemping to install Logstash:
```
E: Conflicting values set for option Signed-By regarding source https://artifacts.elastic.co/packages/6.x/apt/ stable: /usr/share/keyrings/elasticsearch-keyring.gpg !=
E: The list of sources could not be read.
```
Solved this by removing the extra `deb https://artifacts.elastic.co/packages/6.x/apt stable main` line(s) from file `/etc/apt/sources.list.d/elastic-6.x.list` and running the update and install again.

4. Create a symlink to enable Logstash to automatically start on system boot
```
sudo systemctl enable logstash
```
5. Start Logstash
```
sudo systemctl start logstash
```
Check if Logstash process is running
```
ps -ef|grep logstash
```
Output should look like this:
```
logstash    4099       1 98 09:02 ?        00:00:30 /bin/java -Xms1g -Xmx1g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djruby.compile.invokedynamic=true -Djruby.jit.threshold=0 -Djruby.regexp.interruptible=true -XX:+HeapDumpOnOutOfMemoryError -Djava.security.egd=file:/dev/urandom -cp /usr/share/logstash/logstash-core/lib/jars/animal-sniffer-annotations-1.14.jar:/usr/share/logstash/logstash-core/lib/jars/commons-codec-1.11.jar:/usr/share/logstash/logstash-core/lib/jars/commons-compiler-3.0.8.jar:/usr/share/logstash/logstash-core/lib/jars/error_prone_annotations-2.0.18.jar:/usr/share/logstash/logstash-core/lib/jars/google-java-format-1.1.jar:/usr/share/logstash/logstash-core/lib/jars/gradle-license-report-0.7.1.jar:/usr/share/logstash/logstash-core/lib/jars/guava-22.0.jar:/usr/share/logstash/logstash-core/lib/jars/j2objc-annotations-1.1.jar:/usr/share/logstash/logstash-core/lib/jars/jackson-annotations-2.9.10.jar:/usr/share/logstash/logstash-core/lib/jars/jackson-core-2.9.10.jar:/usr/share/logstash/logstash-core/lib/jars/jackson-databind-2.9.10.1.jar:/usr/share/logstash/logstash-core/lib/jars/jackson-dataformat-cbor-2.9.10.jar:/usr/share/logstash/logstash-core/lib/jars/janino-3.0.8.jar:/usr/share/logstash/logstash-core/lib/jars/javassist-3.22.0-GA.jar:/usr/share/logstash/logstash-core/lib/jars/jruby-complete-9.2.7.0.jar:/usr/share/logstash/logstash-core/lib/jars/jsr305-1.3.9.jar:/usr/share/logstash/logstash-core/lib/jars/log4j-api-2.17.1.jar:/usr/share/logstash/logstash-core/lib/jars/log4j-core-2.17.1.jar:/usr/share/logstash/logstash-core/lib/jars/log4j-slf4j-impl-2.17.1.jar:/usr/share/logstash/logstash-core/lib/jars/logstash-core.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.commands-3.6.0.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.contenttype-3.4.100.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.expressions-3.4.300.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.filesystem-1.3.100.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.jobs-3.5.100.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.resources-3.7.100.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.core.runtime-3.7.0.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.equinox.app-1.3.100.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.equinox.common-3.6.0.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.equinox.preferences-3.4.1.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.equinox.registry-3.5.101.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.jdt.core-3.10.0.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.osgi-3.7.1.jar:/usr/share/logstash/logstash-core/lib/jars/org.eclipse.text-3.5.101.jar:/usr/share/logstash/logstash-core/lib/jars/slf4j-api-1.7.25.jar org.logstash.Logstash --path.settings /etc/logstash
root        4125    2293  0 09:03 pts/1    00:00:00 grep --color=auto logstash
```
7. (Not sure if really needed?) Add ports 9200 and 5044 to the allowed ports to open for incoming traffic
```
sudo firewall-cmd --add-port=9600/tcp --zone=public --permanent
sudo firewall-cmd --add-port=5044/tcp --zone=public --permanent
```
Reload configuration after adding the ports
```
firewall-cmd --reload
```
To check list of allowed ports:
```
sudo firewall-cmd --list-ports
```

## Test Logstash installation
Run the most basic Logstash pipeline to test
```
cd /usr/share/logstash
bin/logstash -e 'input { stdin { } } output { stdout {} }'
```
The pipeline is running once the following messages are output (may take a bit of time):
```
[INFO ] 2022-09-19 12:12:17.008 [Converge PipelineAction::Create<main>] pipeline - Pipeline started successfully {:pipeline_id=>"main", :thread=>"#<Thread:0xe4a88ac run>"}
The stdin plugin is now waiting for input:
[INFO ] 2022-09-19 12:12:17.143 [Ruby-0-Thread-1: /usr/share/logstash/lib/bootstrap/environment.rb:6] agent - Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
[INFO ] 2022-09-19 12:12:17.675 [Api Webserver] agent - Successfully started Logstash API endpoint {:port=>9600}
```
At this point you can enter any message to test and the reply should look something like this:
```
/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/awesome_print-1.7.0/lib/awesome_print/formatters/base_formatter.rb:31: warning: constant ::Fixnum is deprecated
{
      "@version" => "1",
          "host" => "ip-10-255-255-252",
    "@timestamp" => 2022-09-19T12:12:53.025Z,
       "message" => "Testing 123"
}
```
`ctrl + c` to exit

To kill a Logstash instance, use the `ps -ef|grep logstash` command to find the process ID and then command `kill -9 <PID>` to terminate the instance
