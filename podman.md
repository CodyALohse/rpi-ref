# Podman

## Creating a pod

```bash
podman pod create --name=your-pod-name

podman create --pod=your-pod-name --name=container-a
```


## Systemd - auto restart pod
Generate the systemd files

```
cd $HOME./config/systemd/user
podman generate systemd --new --files --name your-pod-name
```

## Allow non-root user to auto start systemd services

```
loginctl enable-linger <username>
```
