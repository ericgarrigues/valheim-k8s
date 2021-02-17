# valheim-k8s

kubernetes deployment for a valheim game-server. Based off the dockerization work by lloesche [here](https://github.com/lloesche/valheim-server-docker).

## Usage

Note: This assumes the node you are running on can use `/data/valheim` mounted as a host volume for persistence.  

```bash
helm repo add valheim-k8s https://addyvan.github.io/valheim-k8s/
helm repo update
helm install valheim-server valheim-k8s/valheim-k8s  \
  --set worldName=example-world-name \
  --set serverName=example-server-name \
  --set password=password \
  --set storage.kind=hostvol \
  --set storage.hostvol.path=/data/valheim
```

## Configuration

| Parameter                                  | Description                                                | Default                           |
|:-------------------------------------------|:-----------------------------------------------------------|:----------------------------------|
| `worldName`                                | Prefix of the world files to use (will make new if missing)| `example-world-name`              |
| `serverName`                               | Server name displayed in the server browser(s)             | `example-server-name`             |
| `password`                                 | Server password                                            | `password`                        |
| `storage.kind`                             | Storage strategy/soln used to provide the game-server with persistence | `hostvol`             |
| `storage.hostvol.path`                     | The folder to be mounted into /config in the game-server pod | `/data/valheim`                 |
| `networking.serviceType`                   | The type of service e.g `NodePort`, `LoadBalancer` or `ClusterIP` | `LoadBalancer`                 |

## Persistence

The only form of persistence currently available is through mounting a `hostvol`. Please create an issue if you would like support for specific cloud storage solutions via PVCs / storageclasses. They vary by provider so PRs / testers welcome for this. 

### Using a Host Volume

On the node you wish to use make sure the folder you are mounting exists (ideally empty if you are starting a new world). Once you spin up the game pod you should see the following files created:
```bash
$ ls /data/valheim
adminlist.txt  backups  bannedlist.txt  permittedlist.txt  prefs  worlds
```

### Using an existing world

To use an existing world simply set the `worldName` parameter to the name of your world then save the `.db` and `.fwl` files to the directory mounted into the pod. For example, if your world is named `myworld` then set `worldName: myworld` in your values file (or `--set worldName=myworld`) and assuming you are mounting at `/data/valheim` then your directory should look like: 
```bash
$ ls /data/valheim/worlds/
myworld.db  myworld.fwl
```

## Connecting to your world

Assuming you have taken care of the networking (port-forwarding if needed, LoadBalancer IP is created, ...): 
* In the steam UI (NOT IN GAME) go to view->servers->add favorite
  * set the port to 2457 (will randomly change to 2456 in the UI but ignore that)
* To connect double click the server in the steam servers explorer
  * You will be asked for the password in the steam ui and in game

## Potential future updates

If there is interest I can hash out the following:
* Cronjob to save backups to s3, blob storage, or minio
  * Then clear up the space in the hostvol/pvc
* More persistence options, namely for those looking to run this on the cloud
  * I'm familiar with Azure and AWS but I'm sure GCP and others will be fairly simple to figure out if we have testers
