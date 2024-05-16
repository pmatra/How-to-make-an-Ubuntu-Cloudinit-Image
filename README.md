# How to create ubuntu cloudinit image

<h3><strong>Cloud init images are great for ProxMox Templates and for launching machines via Terraform.&nbsp;</strong></h3>
<p><strong>Ubuntu images are available here:</strong>&nbsp;&nbsp;<a href="https://cloud-images.ubuntu.com/" target="_blank" rel="noreferrer noopener"></a><a href="https://cloud-images.ubuntu.com">https://cloud-images.ubuntu.com</a></p>
<p><span class="enlighter-text">wget https:</span><span class="enlighter-c0">//cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img</span></p>
<p><strong>You need to install libguestfstools to customize the image.</strong></p>
<p><code><span class="enlighter-text">sudo apt update -y </span><span class="enlighter-g0">&amp;&amp;</span><span class="enlighter-text"> sudo apt install libguestfs-tools -y</span></code></p>
<p>&nbsp;<strong>Once the tools are installed you can start customizing your image:</strong></p>
<p><code><span class="enlighter-text">sudo virt-customize -a focal-server-cloudimg-amd64.</span><span class="enlighter-m3">img</span><span class="enlighter-text"> --install qemu-guest-agent<br /></span>sudo virt-customize -a focal-server-cloudimg-amd64.img --run-command 'useradd pmatra'<br />sudo virt-customize -a focal-server-cloudimg-amd64.img --run-command 'mkdir -p /home/pmatra/.ssh'<br />sudo virt-customize -a focal-server-cloudimg-amd64.img --ssh-inject pmatra:file:/home/pmatra/.ssh/authorized_keys<br />sudo virt-customize -a focal-server-cloudimg-amd64.img --run-command 'chown -R pmatra:pmatra /home/pmatra'<br />sudo virt-customize -a focal-server-cloudimg-amd64.img --root-password password:PASSWORD<br />sudo virt-customize -a focal-server-cloudimg-amd64.img --run-command 'echo /etc/sudoers &gt;&gt; pmatra ALL=(ALL) NOPASSWD:ALL'</code></p>
<p>After all of that is done if you want to load it into Proxmox as a template, you need to upload it to your Proxmox Server and run this code.</p>
<p><code>qm create 9000 --name "ubuntu20-cloudinit-template" --memory 2048 --cores 2 --net0 virtio,bridge=vmbr0<br />qm importdisk 9000 focal-server-cloudimg-amd64.img local-lvm<br />qm set 9000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-9000-disk-0<br />qm set 9000 --boot c --bootdisk scsi0<br />qm set 9000 --ide2 local-lvm:cloudinit<br />qm set 9000 --serial0 socket --vga serial0<br />qm set 9000 --agent enabled=1<br />qm template 9000</code></p>
<p>
If you want to clone the image manually you can do it this way via ProxMox Shell or SSH  
</p>
<code>sudo qm clone 9000 999 --name test-clone-cloud-init
sudo qm set 999 --ipconfig0 ip=10.98.1.96/24,gw=10.98.1.1
sudo qm start 999</code>