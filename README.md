# Install Elastisearch And Kibana


### 1. Download the Elasticsearch installer and Kibana binaries.

**Elasticsearch:**

https://www.elastic.co/guide/en/elasticsearch/reference/current/windows.html

https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.12.1.msi

**Kibana:**

https://www.elastic.co/pt/downloads/kibana

https://artifacts.elastic.co/downloads/kibana/kibana-7.12.1-windows-x86_64.zip

### 2. Run Elasticsearch installer with base settings. The installer installs the binaries and a windows service with the name "Elasticsearch".

### 3. Unzip the Kibana zip to a directory of your choice.

![1](https://user-images.githubusercontent.com/6674269/119868191-a72f1b00-bf16-11eb-9fdb-5a79d63cfc22.png)

### 4. Elasticsearch is already installed as a windows service, Kibana is necessary to do the installation:

- Download the NSSM (Not-Sucking Service Manager) https://nssm.cc/;
- Place NSSM.exe in a directory of your choice;

![2](https://user-images.githubusercontent.com/6674269/119868217-aeeebf80-bf16-11eb-99c9-706b9a6bd573.png)

- Use cmd command line until the location of nssm.exe: cd "C:\Program Files\nssm";
- Run the command line nssm install Kibana. The nssm GUI will open;
- Fill with the following options (path D:\elasticsearch kibana\bin\kibana.bat) and install;

![3](https://user-images.githubusercontent.com/6674269/119868261-ba41eb00-bf16-11eb-9c3d-a8c867a32286.png)
![4](https://user-images.githubusercontent.com/6674269/119868269-bc0bae80-bf16-11eb-9d84-6c7b2c5544de.png)
![5](https://user-images.githubusercontent.com/6674269/119868273-bd3cdb80-bf16-11eb-8356-53d0af7b8ee9.png)


### 5. Both services are running and, by definition, Elasticsearch runs on port 9200 of localhost and Kibana on 5601.

1. http://localhost:9200
1. http://localhost:5601

### 6. To expose these two urls off the server, it is necessary to configure a reverse proxy for each of the localhost on iis.

- Install the Web Platform Installer;
- In the Web Platform Installer, install the (ARR) Application Routing Request;

![6](https://user-images.githubusercontent.com/6674269/119868286-c29a2600-bf16-11eb-8af8-c4f7cf9c5ab2.png)

- Create websites on IIS;

![7](https://user-images.githubusercontent.com/6674269/119868332-d0e84200-bf16-11eb-811e-a7aebdc59c32.png)

- On the chosen website, select URL Rewrite;

![8](https://user-images.githubusercontent.com/6674269/119868350-d776b980-bf16-11eb-9eb2-c8778ccea104.png)

- Add New Rule and Reverse Proxy;

![9](https://user-images.githubusercontent.com/6674269/119868382-e1002180-bf16-11eb-9677-3cf5132f7bb6.png)

- Fill in with the localhost of the chosen application and finish;

![10](https://user-images.githubusercontent.com/6674269/119868392-e52c3f00-bf16-11eb-90a5-8c8b29cd120b.png)

- http://127.0.0.1,5601 for Kibana and http://127.0.0.1:9200 for Elasticsearch;

- This rule is configured in the web.config of the two websites;

Elasticsearch

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <staticContent>
            <mimeMap fileExtension="." mimeType="application/octet-stream" />
        </staticContent>
        <rewrite>
            <rules>
                <rule name="ReverseProxyInboundRule1" stopProcessing="true">
                    <match url="(.*)" />
                    <action type="Rewrite" url="http://127.0.0.1:9200/{R:1}" />
                </rule>
            </rules>
        </rewrite>
		<httpProtocol>
			<customHeaders>
				<add name="Access-Control-Allow-Origin" value="*" />
				<add name="Access-Control-Allow-Headers" value="Content-Type" />
				<add name="Access-Control-Allow-Methods" value="GET, POST, PUT, DELETE, OPTIONS" />
			</customHeaders>
		</httpProtocol>
    </system.webServer>
</configuration>

```
Kinaba

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <staticContent>
            <mimeMap fileExtension="." mimeType="application/octet-stream" />
        </staticContent>
        <rewrite>
            <rules>
                <rule name="ReverseProxyInboundRule2" stopProcessing="true">
                    <match url="(.*)" />
                    <action type="Rewrite" url="http://127.0.0.1:5601/{R:1}" />
                </rule>
            </rules>
        </rewrite>
		<httpProtocol>
			<customHeaders>
				<add name="Access-Control-Allow-Origin" value="*" />
				<add name="Access-Control-Allow-Headers" value="Content-Type" />
				<add name="Access-Control-Allow-Methods" value="GET, POST, PUT, DELETE, OPTIONS" />
			</customHeaders>
		</httpProtocol>
    </system.webServer>
</configuration>


```


![11](https://user-images.githubusercontent.com/6674269/119868415-ebbab680-bf16-11eb-8bb9-1d2f09798567.png)


### 7. Authentication in Kibana is configured later.

### 8. Access to Kibana was configured in IIS at url xxxxx.

![12](https://user-images.githubusercontent.com/6674269/119868430-ef4e3d80-bf16-11eb-80f0-dea257da702e.png)


### 9. Authentication in Elasticsearch (Basic Authentication).

- Stop Elasticsearch and Elasticsearch Kibana windows services;
- Go to elasticsearch.yml configuration file (D:\elasticsearch\Elastic\ElasticSearch\config);
- Change configuration to xpack.security.enabled: true and discovery.type: single-node;
- Restart windows services;
- Open CMD and go to directory D:\elasticsearch\Elastic\7.12.1\bin;
- Run elasticsearch-setup-passwords interactive;


![13](https://user-images.githubusercontent.com/6674269/119868443-f2e1c480-bf16-11eb-9590-9e260f3b5719.png)

![14](https://user-images.githubusercontent.com/6674269/119868460-f7a67880-bf16-11eb-8ad6-a951ab39871e.png)


### 10. Go to config file kibana.yml (D:\elasticsearch kibana\config).

- Set up:

1. elasticsearch.username: "kibana_system"
1. elasticsearch.password: "xxx"
1. xpack.security.encryptionKey: "xxx"
1. xpack.security.session.idleTimeout: "1h"
1. xpack.security.session.lifespan: "30d"

![15](https://user-images.githubusercontent.com/6674269/119868483-fc6b2c80-bf16-11eb-96b7-fa86a12451b5.png)


### 11. Enter kibana with username elastic.



