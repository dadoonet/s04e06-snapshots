# Demo script used for Elastic Daily Bytes S04E06 - Cloud - Snapshots

![Cloud - Snapshots](images/00-talk.png "Cloud - Snapshots")

## Setup

You first need to [create a Google Cloud Storage bucket](https://www.elastic.co/guide/en/elasticsearch/reference/8.1/repository-gcs.html#repository-gcs-creating-bucket) to hold the snapshots. It must be named `elastic-daily-bytes`.
Then [create a service account](https://www.elastic.co/guide/en/elasticsearch/plugins/current/repository-gcs-usage.html#repository-gcs-using-service-account) and download the json service file to `elasticsearch-config/service-account.json`.
You need to create a first deployment, let say using version 7.17.3, in Google Cloud, in Belgium. Once ready, initialize it with the Sample Web Logs demo.
Create a second deployment, let say using version 8.1.3, in Azure, in Paris. Once ready, go to the security tab and remove the `gcs.client.demo.credentials_file` security key.
Open Kibana and remove the `elastic-daily-bytes` repository and the `person` index.

## Pre-run

* Open [ELastic Cloud](https://cloud.elastic.co/) with the personal account.
* Open [Elastic GCS plugin documentation](https://www.elastic.co/guide/en/elasticsearch/reference/8.1/repository-gcs.html#repository-gcs-creating-bucket)
* Open [elastic-daily-bytes bucket configuration](https://console.cloud.google.com/storage/browser/elastic-daily-bytes;tab=configuration?prefix=&forceOnObjectsSortingFiltering=false)

## Run

Snapshots are automatically run in the background as soon as you create a deployment.

![Cloud - Snapshots](images/10-existing-snapshots.png "Cloud - Snapshots")

Open Snapshot Management. And explain the tabs:

* Snapshots: every snapshot is here. Look at the dates. They are run every 30 minutes by default. Rollover the restore button to see that you can easily restore one version if something went wrong, like you removed by mistake an index. (This is a true story that happened to me).
* Repositories: the `found-snasphots` is created automatically when you run on cloud. It uses the right cloud storage depending on the cloud provider you chose when you deployed your cluster.

![Cloud - Snapshots](images/11-default-repository.png "Cloud - Snapshots")

![Cloud - Snapshots](images/12-repository-details.png "Cloud - Snapshots")

* Policies: if you want to change how often the snapshot is ran, you can edit the default `cloud-snapshot-policy` and may be backup only every day.

![Cloud - Snapshots](images/13-policies.png "Cloud - Snapshots")

![Cloud - Snapshots](images/14-cloud-snapshot-policy.png "Cloud - Snapshots")

Note that we retain by default up to 100 snapshots and snapshots that needs to be removed must be older than 3 days.

![Cloud - Snapshots](images/15-policy-details.png "Cloud - Snapshots")

### New deployment from a snapshot

We have an old cluster running. (Open "my-old-deployment" Kibana dev console).

```json
GET /kibana_sample_data_logs/_search?track_total_hits=true
```

Let say we want to make sure we can upgrade our cluster. We can create a new deployment from an existing snapshot. Let select a 7.17.3 version and restore from `my-old-deployment`. Then we will be able to upgrade the cluster. We can not directly upgrade to a major version by using an old snapshot for now. It's a bug apparently and should be fixed in the future.

![Cloud - Snapshots](images/21-new-deployment.png "Cloud - Snapshots")

Show that you can change the settings, like the number of nodes...

### Manual repository

We can also use our own repository. This could be useful if you want for example restore a cluster or an index from another deployment which has been deployed in another cloud provider.

For example, we have a new cluster, running 8.1 deployed on Azure in Paris and we want to restore existing data from GCP using Google Cloud Storage.

Let's look at the documentation.

We had already created the GCS bucket (show it).

![Cloud - Snapshots](images/30-gcs-bucket.png "Cloud - Snapshots")

Let's use a service account.
Open IAM & Admin -> Service Account -> elastic-daily-bytes and choose "Manage keys".

![Cloud - Snapshots](images/30-manage-keys.png "Cloud - Snapshots")

Create a new key as JSON.

![Cloud - Snapshots](images/31-add-key.png "Cloud - Snapshots")

Save the file on disk.

Open the "My new deployment in Paris" cloud deployment in the [cloud console](https://cloud.elastic.co/deployments) and open the security tab. Click on `Add settings` near the "Elasticsearch keystore".

Add the setting `gcs.client.demo.credentials_file` as a "JSON/Block file" and copy/paste the content of `elasticsearch-config/service-account.json` in it.
Note that the client name will be `demo`. Then click "Save".

Open Kibana. Go to Stack Management and Snapshot and Restore.

Add a new repository `elastic-daily-bytes`:

![Cloud - Snapshots](images/32-add-gcs-repo.png "Cloud - Snapshots")

With the following settings:

* Client: `demo`. That's part of the setting named we used previoulsy.
* Bucket: `elastic-daily-bytes`.
* Read-only: `true`. We make it read only as we don't want to write to it.

![Cloud - Snapshots](images/33-gcs-settings.png "Cloud - Snapshots")

The repository has been created.

![Cloud - Snapshots](images/34-repo-created.png "Cloud - Snapshots")

Click on "Verify repository":

![Cloud - Snapshots](images/35-verify.png "Cloud - Snapshots")

Click on "2 snapshots found":

![Cloud - Snapshots](images/36-snapshots.png "Cloud - Snapshots")

Restore the `demo-14.47.07*` one. Just select the index `person`:

![Cloud - Snapshots](images/37-restore.png "Cloud - Snapshots")

While this is running, have a look at the other cluster. Open Stack Management and the Upgrade Assistant.

![Cloud - Snapshots](images/38-upgrade-assistant.png "Cloud - Snapshots")

We can launch the upgrade if we want.

Go back to the "My new deployment in Paris" cluster and check that the index `person` is available in the index management.
