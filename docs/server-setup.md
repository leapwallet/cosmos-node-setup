# Server Setup

1. Skip this step if you're using AWS. If you're running as the `root` user, and don't have another user that you can switch to which has `sudo` privileges, then follow these steps:
    1. Create the user:

        ```shell
        read -p 'Enter the user: ' USER
        adduser $USER
        usermod -aG sudo $USER
        exit
        ```
    2. SSH back into the server using the new user.
2. If you're using AWS, set the password:

    ```shell
    sudo passwd ubuntu
    ```
3. Prepare for installation:

    ```shell
    #########################
    ## BEGIN: Install fish ##
    #########################
   
    sudo apt-add-repository -y ppa:fish-shell/release-3
    sudo apt update
    sudo apt -y install fish
    chsh -s /usr/bin/fish

    #######################
    ## END: Install fish ##
    #######################
   
    sudo apt -y full-upgrade
    
    sudo reboot
    ```
4. Set up the [firewall](firewall.md).
5. Set up [Node Exporter](node-exporter-setup.md).
