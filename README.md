Vagrant development environment for the [CoreOS SDK](https://coreos.com/docs/sdk).

You should already have the ubuntu-14.10 base box in your local Vagrant installation (as described at the [From Iso To Vagrant Box](http://blog.ruilopes.com/from-iso-to-vagrant-box.html) article). You should have built it with a decent disk size like:

    packer build -var disk_size=81920 ubuntu-14.10.json
