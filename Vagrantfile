Vagrant.configure("2") do |config|
  ["dc1","dc2"].to_enum.with_index(1).each do |dc, d|
    # dc is the name of dc, dc1 or dc2
    # d is a integer, 1 or 2 

    # datacenter variable (dc1 or dc2)
    datacenter = dc
    # Consul Server IP
    server_ip="192.168.#{50+d}.11"
    # hostname of the Consul server
    hostname_server="server-#{dc}"

    config.vm.define hostname_server do |server|
      server.vm.box = "berchev/xenial64"
      server.vm.hostname = hostname_server
      server.vm.network "private_network", ip: server_ip
      server.vm.provision :shell, path: "scripts/server_provision.sh", env: { "ip_server" => server_ip, "node_name" => hostname_server, "dc" => datacenter }
    end

    # increase/decrease range in order to change the count of the clients.
    (1..2).each do |i|
      client_ip="192.168.#{50+d}.#{20+i}"
      hostname_client="client#{i}-#{dc}"
      tag="v1.#{d}.#{i}"
      config.vm.define hostname_client do |client|
        client.vm.box = "berchev/nginx64"
        client.vm.hostname = hostname_client
        client.vm.network "private_network", ip: client_ip
        client.vm.provision :shell, path: "scripts/client_provision.sh", env: { "ip_client" => client_ip, "ip_server" => server_ip, "node_name" => hostname_client, "dc" => datacenter }
        client.vm.provision :shell, path: "scripts/client_service.sh", env: { "tag" => tag }
      end
    end
  end #dc,d
end