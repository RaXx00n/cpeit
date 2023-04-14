
# <b> CPEit - Common Platform Enumeration Inventory Tools - 2023 CSI Project</b>


CPEit consists of two components under development:

  *An Agent containing scripts that gather information on the software installed on a machine from multiple system sourcs,
  makes an educated guess on potential CPEs, and prepares them for ingestion into Elastic.
  
  *An ELK stack configured for taking the JSON data generated by the Agent and fuzzy string-matching NIST's Official CPE Dictionary for it,
  allowing an administrator to quickly build an inventory of CPEs for all installed software on their environment, and keep it up to date automatically
  by setting the Agent to run on intervals and Elastic to generate alerts.
  
  
## <b>Install the ELK Stack</b>

<b> NOTE: This process has now been automated with the script cpeit-elk/install.sh. Files can be uninstalled and cleaned with the script cpeit-elk/uninstall.sh. The scripts must be run as root. You can then skip to [Configuring the ELK stack for authentication](https://github.com/RaXx00n/cpeit/blob/main/README.md#configure-the-elk-stack-for-authentication). </b>

The manual section below exists for troubleshooting and reference in potential script changes. It is also missing some steps.



### Manual method (reference and incomplete, please use install.sh):

Install Elasticsearch from the repo with:

<code>apt-get install elasticsearch</code>

Test running elasticsearch by navigating to the installation path (the example of /usr/share/elasticsearch/ will be used in this README) 
and run 
<code>./bin/elasticsearch</code>

If you get errors for missing config files, the default [elasticsearch.yml](https://github.com/RaXx00n/cpeit/blob/main/cpeit-elk/elasticsearch.yml) and [log4j2.properties](https://github.com/RaXx00n/cpeit/blob/main/cpeit-elk/log4j2.properties) files can be found in cpeit/cpeit-elk and should be copied to: 
/usr/share/elasticsearch/log4j2.properties
/usr/share/elasticsearch/elasticsearch.yml

You will need to configure elasticsearch.yml with the @path.data@ and @path.logs@ variables. 
/usr/share/elasticsearch/data and /var/log/elasticsearch/ can be used.

Verify Elasticsearch is running by navigating to http://localhost:9200 in your web browser. You should see a JSON response that contains basic information about your installation.

<b>Install Logstash:</b>

<code>apt-get install logstash</code>

Configure [logstash.conf](https://github.com/RaXx00n/cpeit/blob/main/cpeit-elk/logstash.conf) at /usr/share/logstash.conf

Test runnng logstash from the installation directory (by default /usr/share/logstash/) with the command:

<code>./bin/logstash -f /usr/share/logstash/logstash.conf --path.settings /etc/logstash</code>

If you encounter permissions errors you may need to open permissons to /var/lib/logstash/ and /var/log/logstash/

Finally, Logstash will require the Filter Fingerprint plugin. Navigate to /usr/share/logstash/ and run:

<code>./bin/logstash-plugin install logstash-filter-fingerprint</code>

If you get an error about not being able to write to Gemfile you will need to open permissions to Gemfile and Gemfile.lock

Install Kibana from the repo:

<code>apt-get install kibana</code>

Test running Kibana from the installation directory (by default /usr/share/kibana) with the command:
<code>./bin/kibana</code>

If you encounter permissions errors you may need to open permissions to /usr/share/kibana/

If you are missing [Kibana.yml](https://github.com/RaXx00n/cpeit/blob/main/cpeit-elk/kibana.yml), it can be found in cpeit-elk. 
and placed at:
<code>/usr/share/kibana/config/kibana.yml</code> (you may need to create this directory)

With all three components of the ELK stack running, you should be able to go to 127.0.0.1:5601 on your browser and see the Dashboard. Click on Analytics > Discover and verify that your data has been ingested from the JSON file.

## <b>Configure the ELK Stack for authentication</b>

Start logstash with

<code>/usr/share/logstash/bin/logstash -f /usr/share/logstash/logstash.conf --path.settings /etc/logstash</code>

Note the password generated from running:

 <code>/usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic</code>
 
 Copy and paste the password into the password field in /usr/share/logstash/logstash.conf
 
 Note the enrollment token generated from the command below:
 
 <code>/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana</code>
 
 Navigate to 127.0.0.1:5601 and paste this token in, and then run the below for the Kibana verification code:
 
 <code>sudo /usr/share/kibana/bin/kibana-verification-code</code>

This is the code to connect Kibana back to Elasticsearch.

Now all three components of the stack should be working and connected to each other and digesting the default path set in logstash.conf. By default it is
/home/kali/Desktop/testdata.ndjson

testdata.ndjson has been included in the repo
