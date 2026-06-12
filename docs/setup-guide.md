Hướng dẫn cài đặt máy chủ elkMonitoring version 7.x
---------------------------------------------
sudo apt-get update
sudo apt-get upgrade -y
sudo hostnamectl set-hostname elkMonitoring
sudo timedatectl set-timezone Asia/Ho_Chi_Minh
---------------------------
#Cài đặt java 8 jdk
sudo apt install -y openjdk-8-jdk
----------------
#Cài đặt elasticsearch
sudo apt-get install apt-transport-https
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update && sudo apt-get install elasticsearch

#cấu hình elasticsearch
sudo nano /etc/elasticsearch/elasticsearch.yml
# comment cái này #network.host: 192.168.0.1 và thêm dòng này bên dưới network.host
network.bind_host: ["127.0.0.1", "địa_chỉ_ip_elk_server"]
thêm cuối dòng discovery.type: single-node

sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
# kiểm tra trạng thái elasticsearch
sudo systemctl status elasticsearch
#kiểm tra thông tin của elasticsearch
sudo apt-get install -y curl
curl -X GET "localhost:9200"
#kiểm tra các port, ip, giao thức
apt install net-tools
netstat -plntu 
------------------
#Cài đặt Kibana
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update && sudo apt-get install -y kibana
#cấu hình kibana
sudo nano /etc/kibana/kibana.yml
# sửa server.host 0.0.0.0
sudo systemctl enable kibana
sudo systemctl start kibana
#kiểm tra kibana
netstat –plntu 

#chạy thử trong browser: http://192.168.239.132:5601
-----------------
#Cài đặt Logstash
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update && sudo apt-get install -y logstash
# tạo 1 file có tên là 01-logstash.conf
nano /etc/logstash/conf.d/01-logstash.conf
#cấu hình trong file đấy bằng đoạn code sau
input {
  beats {
    port => 5044
  }
}

filter {
  if [fields][log_type] == "apache_access" {
    grok {
      match => { "message" => "%{COMMONAPACHELOG}" }
    }
  }
  
  if [fields][log_type] == "mod_secu" {
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
  }
}

output {
  if [fields][log_type] == "apache_access" {
    elasticsearch {
      hosts => ["http://192.168.239.132:9200"]
      index => "apache_access-%{+YYYY.MM.dd}"
      user => "elastic"
      password => "310102"
    }
  }
  
  if [fields][log_type] == "mod_secu" {
    elasticsearch {
      hosts => ["http://192.168.239.132:9200"]
      index => "mod_secu-%{+YYYY.MM.dd}"
      user => "elastic"
      password => "310102"
    }
  }
}

#tiến hành enable logstash
sudo systemctl start logstash
sudo systemctl enable logstash

------------------------------
Đến đây, ta đã cài đặt và cấu hình xong elk_stack
_____________________________________________________________________________________________________________________________________
Tiếp đến, ta cài đặt beats để gửi log lên elk server. Việc này tùy các bạn lựa chọn
mô hình elk. Dưới đây mình chọn cài filebeat ở máy client để gửi log từ client về elk server
----------------------------------

#Tiếp theo cấu hình filebeat ở client1
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update
sudo apt-get install -y filebeat
#truy cập vào file config để cấu hình
sudo nano /etc/filebeat/filebeat.yml
# sửa enable phần filebeat input thành true, type: log, path: - /var/log/secure và - /var/log/messages - /var/log/suricata/  (trước đó là - /var/log/*.log)

#cấu hình đầu ra trong setup.kibana: ---(bỏ)
#host: "192.168.239.130:5601" ---(bỏ)
#cấu hình đầu ra trong output.elasticsearch: ---(bỏ)
#hosts: ["192.168.239.130:9200"] ---(bỏ)

#test đầu ra của filebeat
filebeat test output
systemctl enable filebeat 
filebeat modules enable system
filebeat modules enable suricata
#kiem tra danh sach module
filebeat modules list
# Sau đó ta load dashboard và index template cho kibana với lệnh
filebeat setup –e ---(bỏ)
# quay lại elkMonitoring để test
curl -X GET 'http://localhost:9200/filebeat-*/_search?pretty' ---(bỏ)


___________________________________________________________________________________________________________________________________________
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update

sudo systemctl status docker
sudo apt update
sudo apt install docker-compose
sudo apt update
sudo apt install apt-utils


sudo docker-compose up -d

sudo docker-compose ps

#dùng khi khởi động lại
sudo docker-compose down (bỏ)
sudo docker-compose up --build (bỏ)
or
sudo docker-compose up --build -d (bỏ)

docker start $(docker ps -a -q)


#gỡ
#dừng lại tất cả container đang chạy
docker stop $(docker ps -a -q)

#Xóa tất cả các container đã dừng
docker rm $(docker ps -a -q)

#Xóa các hình ảnh Docker
docker rmi $(docker images -q)

____________________________________

#cài đặt ssh
sudo apt update
sudo apt upgrade
sudo apt install openssh-server
sudo systemctl status ssh
sudo systemctl enable ssh

gedit /etc/ssh/sshd_config
#sửa:
PasswordAuthentication yes
PermitRootLogin yes
Port xxxx

sudo systemctl restart ssh

ssh-keygen -R 192.168.239.129
ssh-keygen -R [192.168.239.129]:2229
ssh-keygen -R [192.168.239.130]:2230


____________________________________

#cấu hình theo dõi lưu lượng truy cập web
#trên máy ubuntu-client
docker exec -it WEB /bin/bash
filebeat modules enable apache

nano /etc/filebeat/filebeat.yml
#thêm

filebeat.inputs:
- type: log
  enabled: true
  id: apache-access-log
  paths:
    - /var/log/apache2/access.log
  fields:
    log_type: apache_access

- type: log
  enabled: true
  id: modsecurity-error-log
  paths:
    - /var/log/apache2/error.log
  fields:
    log_type: mod_secu


output.logstash:
  hosts: ["192.168.239.132:5044"]

---=------
#dành cho docker
filebeat test config
filebeat test output
service filebeat restart
service filebeat status
docker restart WEB (nếu tuyệt vọng quá)
------
#dành cho ubuntu (không cần)
sudo filebeat test config
filebeat test output
sudo systemctl restart filebeat



_________________________
#trên máy elkMonitoring

nano /etc/elasticsearch/elasticsearch.yml
#thêm vào cuối dòng:
xpack.security.enabled: true

systemctl restart elasticsearch
/usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive
#tiến hành nhập mật khẩu các loại


nano /etc/kibana/kibana.yml
#sửa
elasticsearch.hosts: ["http://192.168.239.132:9200"]
elasticsearch.username: "elastic"
elasticsearch.password: "your_password_here"
sudo systemctl restart kibana

sudo nano /etc/logstash/conf.d/01-logstash.conf (bỏ)

sudo systemctl restart logstash (bỏ)

sudo tail -f /var/log/logstash/logstash-plain.log (bỏ)

_____________________________________________________
sudo systemctl stop gdm3
sudo systemctl disable gdm3
sudo reboot
sudo systemctl start gdm3

__________________________________________

#cấu hình waf
nano modsec_rules.conf

#thêm
Include "/etc/apache2/modsecurity.d/modsecurity.conf"
Include "/etc/apache2/modsecurity.d/owasp-crs/crs-setup.conf"
Include "/etc/apache2/modsecurity.d/owasp-crs/rules/*.conf"

nano 000-default.conf
#thêm
<VirtualHost *:80>
	modsecurity on
	modsecurity_rules_file /etc/apache2/modsecurity.d/modsec_rules.conf 
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html
	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

nano Dockerfile

# Use Ubuntu as the base image
FROM ubuntu:20.04

# Set noninteractive mode and configure time zone
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Ho_Chi_Minh

# Install Required Dependencies
RUN apt-get update -y && \
    apt-get install -y g++ flex bison curl apache2-dev \
        doxygen libyajl-dev ssdeep liblua5.2-dev \
        libgeoip-dev libtool dh-autoreconf \
        libcurl4-gnutls-dev libxml2 libpcre++-dev \
        libxml2-dev git wget tar apache2 php \
        libapache2-mod-php php-mysql nano apt-transport-https gnupg2 \
        autoconf automake pkg-config && \
    wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add - && \
    echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee -a /etc/apt/sources.list.d/elastic-7.x.list && \
    apt-get update && \
    apt-get install -y filebeat packetbeat libpcap0.8 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Download LibModsecurity 
RUN git clone https://github.com/owasp-modsecurity/ModSecurity.git

# Initialize and update git submodules to get libInjection
RUN cd ModSecurity && \
    git submodule init && \
    git submodule update

# Compile and Install LibModsecurity
RUN cd ModSecurity && \
    ./build.sh && ./configure && \
    make && make install

# Install ModSecurity-Apache Connector
RUN cd ~ && git clone https://github.com/SpiderLabs/ModSecurity-apache

RUN cd ~/ModSecurity-apache && \
    ./autogen.sh && \
    ./configure --with-libmodsecurity=/usr/local/modsecurity/ && \
    make && \
    make install

# Load the Apache ModSecurity Connector Module
RUN echo "LoadModule security3_module /usr/lib/apache2/modules/mod_security3.so" >> /etc/apache2/apache2.conf

# Configure ModSecurity
RUN mkdir /etc/apache2/modsecurity.d && \
    cp ModSecurity/modsecurity.conf-recommended /etc/apache2/modsecurity.d/modsecurity.conf && \
    cp ModSecurity/unicode.mapping /etc/apache2/modsecurity.d/ && \
    sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/' /etc/apache2/modsecurity.d/modsecurity.conf
ADD modsec_rules.conf /etc/apache2/modsecurity.d/

# Install OWASP ModSecurity Core Rule Set (CRS) on Ubuntu
RUN git clone https://github.com/SpiderLabs/owasp-modsecurity-crs.git /etc/apache2/modsecurity.d/owasp-crs && \
    cp /etc/apache2/modsecurity.d/owasp-crs/crs-setup.conf.example /etc/apache2/modsecurity.d/owasp-crs/crs-setup.conf

# Activate ModSecurity
RUN mv /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/000-default.conf.old
ADD 000-default.conf /etc/apache2/sites-available/

# Set working directory
WORKDIR /var/www/html/
RUN rm -rf index.html
COPY ./WEB .

# Change permissions
RUN chown -R www-data:www-data /var/www/html/images/
RUN chmod -R 777 /var/www/html/images/

# Expose port 80 to the outside world
EXPOSE 80

# Start Apache server
CMD ["apache2ctl", "-D", "FOREGROUND"]

#tắt quy tắc đánh dấu bất kỳ yêu cầu nào có Hosttiêu đề chứa địa chỉ IP số thay vì tên miền
nano /etc/apache2/modsecurity.d/modsecurity.conf

#thêm
SecRuleRemoveById 920350


_______________________________________

nano /etc/logstash/conf.d/01-logstash.conf

input {
  beats {
    port => 5044
  }
}

filter {
  if [fields][log_type] == "apache_access" {
    grok {
      match => { "message" => ["%{IPORHOST:clientip} - %{DATA:username} \\[%{HTTPDATE:http_date}\\] \"%{WORD:method} %{DATA:path} HTTP/%{NUMBER:apache_http_version}\" %{NUMBER:code} %{NUMBER:sent_bytes}( \"%{DATA:referrer}\")?( \"%{DATA:agent}\")?",
        "%{IPORHOST:clientip} - %{DATA:username} \\[%{HTTPDATE:http_date}\\] \"-\" %{NUMBER:code} -" ] }
    }
    mutate {
      rename => {
        "clientip" => "apache_remote_ip"
        "username" => "apache_user"
        "http_date" => "apache_access_time"
        "method" => "apache_method"
        "path" => "apache_path"
        "code" => "apache_code"
        "apache_http_version" => "apache_http_version"
        "sent_bytes" => "apache_sent_bytes"
        "referrer" => "apache_referrer"
        "agent" => "apache_agent"
      }
    }
  }
  
  if [fields][log_type] == "mod_secu" {
    grok {
      match => { "message" => "(?<event_time>%{MONTH} %{MONTHDAY} %{TIME} %{YEAR})\\] \\[:%{LOGLEVEL:log_level}.*client %{IPORHOST:src_ip}:\\d+\\] (?<alert_message>.*)" }
    }
    # Extract Rules File from Alert Message
    grok {
      match => { "alert_message" => "(?<rulesfile>\\[file \"(/.+.conf)\"\\])" }
    }	
    grok {
      match => { "rulesfile" => "(?<rules_file>/.+.conf)" }
    }	
    # Extract Attack Type from Rules File
    grok {
      match => { "rulesfile" => "(?<attack_type>[A-Z]+-[A-Z][^.]+)" }
    }	
    # Extract Rule ID from Alert Message
    grok {
      match => { "alert_message" => "(?<ruleid>\\[id \"(\\d+)\"\\])" }
    }	
    grok {
      match => { "ruleid" => "(?<rule_id>\\d+)" }
    }
    # Extract Attack Message (msg) from Alert Message 	
    grok {
      match => { "alert_message" => "(?<msg>\\[msg \\\\S(.*?)\\\"\\])" }
    }	
    grok {
      match => { "msg" => "(?<alert_msg>\\\"(.*?)\\\")" }
    }
    # Extract the User/Scanner Agent from Alert Message	
    grok {
      match => { "alert_message" => "(?<scanner>User-Agent' \\\\SValue: `(.*?)')" }
    }	
    grok {
      match => { "scanner" => "(?<user_agent>: (.*?)\\')" }
    }	
    grok {
      match => { "alert_message" => "(?<agent>User-Agent: (.*?)\\')" }
    }	
    grok {
      match => { "agent" => "(?<user_agent>: (.*?)\\')" }
    }	
    # Extract the Target Host
    grok {
      match => { "alert_message" => "(hostname \"%{IPORHOST:dst_host})" }
    }	
    # Extract the Request URI
    grok {
      match => { "alert_message" => "(uri \"%{URIPATH:request_uri})" }
    }
    grok {
      match => { "alert_message" => "(?<ref>referer: (.*))" }
    }	
    grok {
      match => { "ref" => "(?<referer> (.*))" }
    }
    mutate {
      # Remove unnecessary characters from the fields.
      gsub => [
        "alert_msg", "[\"]", "",
        "user_agent", "[:\"'`]", "",
        "user_agent", "^\\s*", "",
        "referer", "^\\s*", ""
      ]
      # Remove the Unnecessary fields so we can only remain with
      # General message, rules_file, attack_type, rule_id, alert_msg, user_agent, hostname (being attacked), Request URI and Referer. 
      remove_field => [ "alert_message", "rulesfile", "ruleid", "msg", "scanner", "agent", "ref" ]
    }
  }
}

output {
  if [fields][log_type] == "apache_access" {
    elasticsearch {
      hosts => ["https://elasticsearch.local:9200"]
      index => "apache_access-%{+YYYY.MM.dd}"
      user => "elastic"
      password => "Administrator"
      ssl => true
      cacert => "/etc/logstash/ca.crt"
    }
  }
  
  if [fields][log_type] == "mod_secu" {
    elasticsearch {
      hosts => ["https://elasticsearch.local:9200"]
      index => "mod_secu-%{+YYYY.MM.dd}"
      user => "elastic"
      password => "Administrator"
      ssl => true
      cacert => "/etc/logstash/ca.crt"
    }
  }
}

-----
nano /etc/filebeat/filebeat.yml

filebeat.inputs:
- type: log
  enabled: true
  id: apache-access-log
  paths:
    - /var/log/apache2/access.log
  fields:
    log_type: apache_access

- type: log
  enabled: true
  id: modsecurity-error-log
  paths:
    - /var/log/apache2/error.log
  fields:
    log_type: mod_secu

output.logstash:
  hosts: ["192.168.239.132:5044"]

