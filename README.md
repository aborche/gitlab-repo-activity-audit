# Gitlab Repo Activity Audit

This bundle of templates extends gitlab repo activity audit and put data to elasticsearch/opensearch/logstash

## Getting started

* install filebeat package from https://www.elastic.co/downloads/beats/filebeat. 
  I recommend use packaged version for your system instead a tar arhive.
* copy contents of inputs.d directory to your server /etc/filebeat/inputs.d
* include config files in your filebeat.yml
    ```commandline
    filebeat.config.inputs:
        enabled: true
        path: inputs.d/*.yml
  ```


## Testing
* change your output settings to console
    ```commandline 
    output:
        console:
            pretty: true
    
    ```
* run filebeat from console
    ```commandline
    $ sudo /usr/share/filebeat/bin/filebeat -e -c /etc/filebeat/filebeat.yml

* watch errors and output

## Setting up elasticsearch/opensearch template
* open Dev Tools in kibana/dashboards
* type
    ```text
  POST _index_template/gitlab_logs
  <contents of template/index_mappings.json>
    ```
* and run command block

Another way
* open console
* call curl 
    ```shell
  curl -X POST -H "Content-Type: application/json" -d @template/index_mappings.json http://localhost:9200
  ```

## Production usage
* Change logstash config like
    ```
    input {
      beats {
        port => 5044
      }
    }
    filter {
      if [host][name] {
        mutate {
          rename => [ "host", "beat_hostname" ]
          add_field => { "host" => "%{[beat_hostname][name]}" }
          convert => {"beat_hostname" => "string"}
        }
      }
    }
    output {
      if 'gitlab' in [tags] {
       opensearch {
         index => "gitlab-%{+YYYY.MM.dd}"
         hosts => ["https://xxx.xxx.xxx.xxx:9200"]
         user => "sensoruser"
         password => "sensoruserpass"
         ssl => true
         ssl_certificate_verification => false
       }
     } else {
        opensearch {
          hosts => ["https://xxx.xxx.xxx.xxx:9200"]
          index => "logstash-%{+YYYY.MM.dd}"
          user => "sensoruser"
          password => "sensoruserpass"
          ssl => true
          ssl_certificate_verification => false
        }
      }
    }
    ```
* Restart logstash
* Comment output to console
* Change filebeat output to logstash
    ```text
  output.logstash:
      hosts: ["localhost:5044"]
      enabled: true
      index: gitlab
  ```
* Enable filebeat
    ```shell
  sudo systemctl enable filebeat
    ```
* Run filebeat
    ```shell
  sudo systemctl start filebeat
    ```
* Check filebeat logs for errors
* Check logstash logs for errors
* Open kibana/dashboards and check for new index
* Configure discover over new index and enjoy

PS: For detailed info read https://habr.com/ru/articles/749912/
