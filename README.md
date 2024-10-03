# Open Source SIEM Stack
<br>
The idea behind this project is to facilitate testing of <a href="https://github.com/socfortress">SOCFortress' Open Source SIEM Stack</a>. This is by no means production safe security-wise, 
and only intended to get people started in a lab environment. This project is licensed under GPLv3 which means you can use and modify in it any way you like, so long as you keep it under the same 
licensing scheme. If you don't feel confortable making necessary changes for moving into production you can get in touch with us and we will help you get there.
<br><br>
## Pre-Deployment
<br>
The first step is to build the Custom Wazuh Manager image using the build script provided in wazuh/custom-wazuh-manager, there you will find instructions for the build.
<br><br>
Next you need to create, by whichever means you prefer, the required Wazuh SSL certs and place them in the wazuh/config/wazuh_indexer_ssl_certs directory. I'm providing the official Wazuh cert generating
script under wazuh/generate-indexer-certs.yml. Instructions for for running this container/script can be found in the <a href="https://github.com/wazuh/wazuh-docker">Official Wazuh Docker Repo</a> 
and also under the specific subdirectory. Note: Also copy the root-ca.pem certificate into the graylog/ subdirectory as you will need it in a later step.
<br><br>
After successful build of the Custom Wazuh Image and SSL cert generation, the next step is to modify all the configuration files provided provided under each module's subdirectory, as the ones provided 
are templates taken from each tool's documentation; and also the .env file, which comes pre-populated up to a certain point for your convenience and is located in the directory root, to suit your 
environment and needs. Refer to each tool's documentation or follow <a href="https://www.youtube.com/@taylorwalton_socfortress/videos">Taylor Walton's Youtube Channel</a> for guidance on how to 
configure each tool.
<br><br>
You'll also need to follow the pre-deployment step outlined in the Graylog section. graylog/README.md
<br><br>
Once that is out of the way, you are ready for Deployment.
<br><br>
## Deployment
<br>
You can now safely start all containers by running:
<br><br>
```
docker compose up -d
```
<br><br>
Most of these services will need some tewaking before you are able to integrate them. I will note the project specific configurations, for the rest please follow 
<a href="https://www.youtube.com/@taylorwalton_socfortress/videos">Taylor's Videos</a>.
<br><br>
### Wazuh
<br>
After initial deployment, the Wazuh Dashboard will present an error in the health check for the wazuh-alerts index as this has not been created yet. This error will only be fully fixed after 
successfully integarting Graylog and the Wazuh Indexer. You can go ahead and create any necessary users/roles for the upcoming integrations to work correctly.
<br><br>
### Graylog
<br>
While the container is running, you'll need to access it's console to perform some extra steps in order to add the Wazuh root CA into Graylog's Java Keystore. Run the following command 
to spawn a shell inside the container.
<br><br>
```
docker exec -it graylog bash
```
<br><br>
Once inside Graylog, you'll need to copy the Java Keystore located at /opt/java/openjdk/lib/security/cacerts into the /usr/share/graylog/data/config/ directory, for this you can run:
<br><br>
```
cp /opt/java/openjdk/lib/security/cacerts /usr/share/graylog/data/config/
```
<br><br>
Next you need to import the Wazuh root CA into the keystore, cd into the /usr/share/graylog/data/config/ and run the following command: (change the certificate name and keystore password as needed, 
but note that if the certificate name doesn't match the template, you'll need to modify the docker-compose.yml file accordingly)
<br><br>
```
keytool -importcert -keystore cacerts -storepass changeit -alias wazuh_root_ca -file root_ca.pem
```
<br><br>
You will be prompted to accept this certificate, type "yes" and enter. After this is done Graylog will be able to connect to the Wazuh Indexer.
<br><br>
## Post-Deployment
<br>
After you've reached this point you can take a short break to pat yourself in the back, it's all downhill from here on out.
<br><br>
You'll need to make sure to create any necessary users in each of the tools for the integrations to work correctly. After that has been completed, you can log into CoPilot and start 
provisioning customers.
<br><br>
## Credits
<br>
Huge shoutout to <a href="https://github.com/socfortress">SOCFortress</a> and <a href="https://www.youtube.com/@taylorwalton_socfortress/featured">Taylor Walton</a> for their hard work and 
contributions to the open source community. And of course all credits to the <a href="https://github.com/wazuh">Wazuh Team</a> for their amazing product.
