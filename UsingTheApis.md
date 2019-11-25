# Overview
Most of the ThingsPro Edge APIs can be invoked either API calls or direct method from IoT Hub. Details of the APIs that ThingsPro Edge supports can be found on [ThingsPro Edge API Doc](https://thingspro-edge.moxa.online/latest/).

In this document, we are going to show how to do this from command line (curl) and IoT Portal. Let's take the API `PUT /system/sshserver` as an example.

In the following section, we are assuming that ThingsPro Edge device is using the ip address `192.168.1.1`. Https is default enabled on the port `8443`.

## Command Line (Curl)
From the prespective of security, API credential is required if we are going to invoke API calls. There are two ways to obtain the API credential.

1. If user is able to ssh into the device, you can get the token from file `/var/thingspro/data/mx-api-token`
    ```
    curl -X PUT https://127.0.0.1:8443/api/v1/system/sshserver \
    -H "Content-Type:application/json" \
    -H "mx-api-token: $(cat /var/thingspro/data/mx-api-token)"
    -d '{"enable":true,"port":22}'
    ```
2. If user is invoking the APIs from remote, you need to get token from the response of `/auth` before doing any further operations.

    ```sh
    curl -X POST https://192.168.1.1:8443/api/v1/auth \
    -H "Content-Type:application/json" \
    -H "mx-api-token: $(cat /var/thingspro/data/mx-api-token)" \
    -d '{"name":"admin","password":"admin@123"}' -k
    ```

    The token can be obtained from `data.token` of the response. Assume the token we retrieved is `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwicm9sZSI6InRlc3QiLCJpYXQiOjE1NjM0MjA5MTQsImV4cCI6NDcxNzAyMDkxNH0.7oGA1VHBingnzeG7hHgWEiZ4JmoserFguyeb7_GUK6E`, then, we can invoke the API call as below.

    ```sh
    curl -X PUT https://192.168.1.1:8443/api/v1/system/sshserver \
    -H "Content-Type:application/json" \
    -H "Authorization:Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwicm9sZSI6InRlc3QiLCJpYXQiOjE1NjM0MjA5MTQsImV4cCI6NDcxNzAyMDkxNH0.7oGA1VHBingnzeG7hHgWEiZ4JmoserFguyeb7_GUK6E" \
    -d '{"enable":true,"port":22}' -k
    ```

## IoT Portal
The generic direct method `thingspro-api-v1` can also be used to invoke APIs from Azure IoT Hub. Note that the default timeout is 10 second, if the timeout isn't adjusted properly, user may get timeout error after invoking direct method.

If user is invoking direct method, there is no need to provide the API credential, since the user should be logged in to Azure account before invoking direct method calls.

Method Name:

```
thingspro-api-v1
```

Payload:
```
{
    "path":"/system/sshserver",
    "method":"PUT",
    "headers":[{"Content-Type":"application/json"}],
    "requestBody":"{\"enable\": true,\"port\": 22}"
}
```