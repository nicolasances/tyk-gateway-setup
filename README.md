# Tyk Gateway installation guide

This project just lists the basic instructions to install the Tyk open source API Gateway using Docker.

*Note that to just use the gateway (without analytics and dev portal), you only need Redis and the Gateway.*

## 1. Create the Docker network

    docker create network tyk-net    

*Note: replace* `tyk-net` *with your network.*

## 2. Start a redis container

    docker run -itd --network tyk-net --name redis redis    

## 3. Create a tyk.conf file

See the tyk configuration file `tyk.conf` in this repository.

Note that the `tyk.conf` file in this example:
 * uses SSL and specifies the certificates to use (in `http_server_options`)
 * set the `listen_port`, but it's **useless** since it's **overridden by the env variable** `TYKLISTENPORT`

## 4. Start the tyk gateway container

    docker run -itd --network tyk-net --name gateway -p 443:443 -e TYKSECRET=asdasd -e TYKLISTENPORT=443 -v /root/tyk-ce/tyk.conf:/opt/tyk-gateway/tyk.conf -v <path-to-certificates-on-host>:/certificates tykio/tyk-gateway

Important steps here:
 * replate `tyk-net` with your network (but has to be the same of redis)
 * set the right port (in this case 8080)
 * set the right `TYKSECRET`: this is going to be used to call the Tyk Gateway own APIs ( `x-tyk-authorization` header)
 * set the right `TYKLISTENPORT`
 * set the right `<path-to-certificates-on-host>` with the folder where the host certificates are contained
 * link the 'tyk.conf' file

*Note that the* `secret` *field in the* `tyk.conf` *file won't actually set the TYK SECRET, since Tyk will override it with the TYKSECRET environment variable: that's why we're passing that env variable in the docker run command.*

*Note that the* `listen_port` *parameter in* `tyk.conf` *is useless and gets overridden by the TYKLISTENPORT env variable that has to be set when starting the container.*

At this point the gateway should be answering on https://<domain>:443/tyk/apis.

Try it with:
    curl https://<domain>:443/tyk/apis -H "x-tyk-authorization: <your TYKSECRET>"    
It should return the list of available released APIs on the gateway.

## 5. Create an API

To create an API on the gateway you can call the following gateway API:

    curl -X POST https://<domain>:443/tyk/apis -H "x-tyk-authorization: <your TYKSECRET>" -H "Content-Type: application/json" -d @expenses.json    

After creating the API, remember to **hot reload** the gateway.

    curl -X GET https://<domain>:443/tyk/reload -H "x-tyk-authorization: <your TYKSECRET>"

An example of api definition file (`expenses.json`) can be found in this repository.
Note that:
 * you have to replace the 'target_url' parameter with the actual target URL where the API is released
 * set 'use_basic_auth' to 'true' to make sure that Basic auth is required on this API

## 6. Create a user on the Gateway

This user is going to be the one used in the Basic Authentication to call the APIs that you are going to deploy on the gateway.

    curl -X POST https://<domain>:443/tyk/keys/<username> -H "x-tyk-authorization: <your TYKSECRET>" -H "Content-Type: application/json" -d @user.json    

`user.json` is a file where you're storing the user definition, **including its password**.

An example of `user.json` can be found in this repository.

Note that:
 * in the `user.json` you have to specify the list of APIs that he can access, which means that **you will have to update the user definition every time you release new APIs that you want him to access**
 * the POST call ends with the username of the user you want to create
