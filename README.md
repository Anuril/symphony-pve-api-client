# PVE2 API wrapper for Symfony

This class provides the building blocks for someone wanting to use PHP to talk to Proxmox's API, and is using symfony anyway.

This is a fork and rewrite of [this](https://github.com/CpuID/pve2-api-php-client). 
This library probably isn't a drop-in replacement for it, but it should be relatively simple to fix your code.

Due to the nature of how the Proxmox VE API works, errors 500 / 501 will return null, otherwise requesting an invalid resource to see if it already exists doesn't work. Ideally, I'd like to work towards 


Relatively simple piece of code, just provides a get/put/post/delete abstraction layer as methods
on top of Proxmox's REST API, while also handling the Login Ticket headers required for authentication.


## More information: 

See http://pve.proxmox.com/wiki/Proxmox_VE_API for information about how this API works.
API spec available at https://pve.proxmox.com/pve-docs/api-viewer/index.html

## Requirements: ##

PHP 8.1+ with symfony

## Usage: ##

Example - Return status array for each Proxmox Host in this cluster.
```
    require_once("./srv/pve2_api.class.php");

    # You can try/catch exception handle the constructor here if you want.
    $pve2 = new PVE2_API("hostname", "username", "realm", "password", port: "port", verify_ssl: true, tokenid: "tokenid", tokensecret: "tokensecret, debug: true);
    # realm above can be pve, pam or any other realm available.
    # if you provide tokenid and tokensecret, you can leave out username & password.
    # debug generates more verbose exceptions. 
    # verify_ssl enables or disables ssl validation. 


    if ($pve2->login()) {
        foreach ($pve2->get_node_list() as $node_name) {
            print_r($pve2->get("/nodes/".$node_name."/status"));
        }
    } else {
        print("Login to Proxmox Host failed.\n");
        exit;
    }
```

Example - Create a new Linux Container (LXC) on the first host in the cluster.

```
    require_once("./pve2-api-php-client/pve2_api.class.php");

    # You can try/catch exception handle the constructor here if you want.
    $pve2 = new PVE2_API("hostname", "username", "realm", "password", port: "port", verify_ssl: true, tokenid: "tokenid", tokensecret: "tokensecret, debug: true);
    # realm above can be pve, pam or any other realm available.

    if ($pve2->login()) {

        # Get first node name.
        $nodes = $pve2->get_node_list();
        $first_node = $nodes[0];
        unset($nodes);

        # Create a Linux Container (LXC) on the first node in the cluster.
        $new_container_settings = array();
        $new_container_settings['ostemplate'] = "local:vztmpl/debian-6.0-standard_6.0-4_amd64.tar.gz";
        $new_container_settings['vmid'] = "1234";
        $new_container_settings['cpus'] = "2";
        $new_container_settings['description'] = "Test VM using Proxmox 2.0 API";
        $new_container_settings['disk'] = "8";
        $new_container_settings['hostname'] = "testapi.domain.tld";
        $new_container_settings['memory'] = "1024";
        $new_container_settings['nameserver'] = "4.2.2.1";

        // print_r($new_container_settings);
        print("---------------------------\n");

        print_r($pve2->post("/nodes/".$first_node."/lxc", $new_container_settings));
        print("\n\n");
    } else {
        print("Login to Proxmox Host failed.\n");
        exit;
    }
```

Example - Modify DNS settings on an existing container on the first host.

```
    require_once("./pve2-api-php-client/pve2_api.class.php");

    # You can try/catch exception handle the constructor here if you want.
    $pve2 = new PVE2_API("hostname", "username", "realm", "password", port: "port", verify_ssl: true, tokenid: "tokenid", tokensecret: "tokensecret, debug: true);
    # realm above can be pve, pam or any other realm available.

    if ($pve2->login()) {

        # Get first node name.
        $nodes = $pve2->get_node_list();
        $first_node = $nodes[0];
        unset($nodes);

        # Update container settings.
        $container_settings = array();
        $container_settings['nameserver'] = "4.2.2.2";

        # NOTE - replace XXXX with container ID.
        var_dump($pve2->put("/nodes/".$first_node."/lxc/XXXX/config", $container_settings));
    } else {
        print("Login to Proxmox Host failed.\n");
        exit;
    }
```

Example - Delete an existing container.

```
    require_once("./pve2-api-php-client/pve2_api.class.php");

    # You can try/catch exception handle the constructor here if you want.
    $pve2 = new PVE2_API("hostname", "username", "realm", "password", port: "port", verify_ssl: true, tokenid: "tokenid", tokensecret: "tokensecret, debug: true);
    # realm above can be pve, pam or any other realm available.

    if ($pve2->login()) {
        # NOTE - replace XXXX with node short name, and YYYY with container ID.
        var_dump($pve2->delete("/nodes/XXXX/lxc/YYYY"));
    } else {
        print("Login to Proxmox Host failed.\n");
        exit;
    }
```

Licensed under the MIT License.
See LICENSE file.
