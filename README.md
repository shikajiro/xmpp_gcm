Android of the log is a project to be able to display in StackDriver of GCP.
Logs are sent to the XMPP server in the GCM, and sends it to the StackDriver Log of GCP.

`Android -> GCM -> XMPP -> StackDriver`

need 
1. create Log Forward Server
2. Android log setting

![](doc/log_sample.png)

#create Log forward Server

## dependencies
### docker
[Install Docker Engine - Docker](https://docs.docker.com/engine/installation/)
### gcloud
[Installing Cloud SDK  |  Cloud SDK Documentation  |  Google Cloud Platform](https://cloud.google.com/sdk/downloads)

## require settings
### if use Google Compute Engine
#### GCP Network
network port tcp:8080 grpc
https://cloud.google.com/solutions/mobile/mobile-compute-engine-grpc

![](doc/fcm.png)

## xmpp start from docker (example GCP)
```
gcloud auth login
docker-machine create -d google \
     --google-project <gcp_project_name> \
     --google-zone asia-northeast1-c \
     --google-machine-type "f1-micro" \
     --google-machine-image https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/debian-8-jessie-v20161027 \
     android_stackdriver_log
eval $(docker-machine env xmpp)
docker run --name android_stackdriver_log -d -e SENDER_ID='your FCM sender id' -e SERVER_KEY='your FCM server key' shikajiro/android_stackdriver_log:0.4

```

Let's start Logging Server !!

### docker log
```
docker logs android_stackdriver_log -f
```

# Android log setting
need fcm(gcm) send process.
## Log Setting Example (if use Timber)

```
    override fun onCreate() {
        super.onCreate()
        Timber.plant(DebugLogTree())
    }

    private class DebugLogTree : Timber.DebugTree() {
        override fun log(priority: Int, tag: String, message: String, t: Throwable?) {
            super.log(priority, tag, message, t)

            if (priority == Log.VERBOSE || priority == Log.DEBUG || priority == Log.INFO) {
                Observable.create<Unit> {
                    val instanceId = FirebaseInstanceId.getInstance().id
                    val fm = FirebaseMessaging.getInstance()
                    // not ui thread.
                    fm.send(RemoteMessage
                            .Builder(SENDER_ID + "@gcm.googleapis.com")
                            .setMessageId(instanceId + DateTime.now().toString("yyyyMMddHHmmss"))
                            .setMessageType("log")
                            .addData("log", message)
                            .addData("instanceId", instanceId)
                            .addData("priority", ""+priority)
                            .addData("tag", tag)
                            .build())
                }.subscribe {

                }
            }
        }
    }
```
