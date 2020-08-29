# Scan-OVAs-with-trivy

An OVA file is a VM image and it is a popular format to package VM images. Tanzu Kubernetes Grid (TKG) releases for vSphere are packaged and shipped as OVAs. These OVAs will contain some of the TKG control plane images (like kube-apiserver, kube-proxy, kube-scheduler etc. Note that additional images will be downloaded from vmware registry during the TKG bootstrap).
Currently, there are two OVAs that you will download when planning to deploy TKG on vSphere 6.7u3 (the ones here are the ones for TKG 1.1.0):

```
photon-3-v1.17.3_vmware.2.ova
photon-3-capv-haproxy-v0.6.3_vmware.1.ova
```
An OVA is a tarball file for a VM image. If you extract it (`tar -zvf photon-3-v1.17.3.2.ova`), you will find it contains a number of files:
```
photon-3-kube-v1.17.3-disk1.ova.vmdk
photon-3-kube-v1.17.3+vmware.2.certificate
photon-3-kube-v1.17.3+vmware.2.mf
photon-3-kube-v1.17.3+vmware.2.ovf
```

The vmdk file will contain the VM file system, so you can create an Ubuntu VM via [VMware Fusion](https://www.vmware.com/uk/products/fusion.html), for example.
Once this is done, you can mount the file system on a specific directory (I called it `tkg-dir`) via `mount /dev/sdb3 /path/to/tkg-dir`.
If you `ls` into the `tkg-dir` directory, you will see the whole file system.

Now, download [trivy CLI](https://github.com/aquasecurity/trivy) following the instructions for Ubuntu.
Once this is done, you can scan the filesystem via `trivy fs /path/to/tkg dir`. Trivy will generate a report that you can save indeed.

An alternative could be to package your filesystem into a Docker image and let [Harbor](https://harbor.io) to scan it. [Harbor](https://harbor.io) cannot scan OVAs as they are VM images because [Harbor](https://harbor.io) can only scan container images. Steps to do so would be to:
1. Tarball your file system: `tar --czvf /path/to/tkg dir`
2. Install [Docker](https://docs.docker.com/engine/install/ubuntu/)
3. Create an image from a tarball: `docker import /path/to/tkg-tarball`
4. Tag your image:
```
docker image ls
```
to pick the IMAGE ID
```
docker tag IMAGE ID <your.harbor.registry.url>/<your-harbor-repo>
```
5. Upload onto Harbor:
`docker login <your.harbor.registry.url> -u <username> -p <password>`
`docker push <your.harbor.registry.url>/<your-harbor-repo>`
Now you can scan the image with [Harbor](https://harbor.io) via Clair. Clair uses trivy for image scanning.

NOTE: an interesting tool to look into container images contents is [dive](https://github.com/wagoodman/dive).
