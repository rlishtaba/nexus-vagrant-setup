SYS_LIBS = %w(wget git make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils).freeze
COMMON_SETUP = <<-EOF
  apt-get update
  apt-get install -y #{SYS_LIBS.join(' ')}
EOF

JVM_SETUP = <<-EOF
apt-get install -y software-properties-common
add-apt-repository -y ppa:webupd8team/java

apt-get update
apt-get -y upgrade
echo debconf shared/accepted-oracle-license-v1-1 select true | debconf-set-selections
echo debconf shared/accepted-oracle-license-v1-1 seen true | debconf-set-selections
apt-get -y install oracle-java8-installer oracle-java8-set-default

echo "JAVA_HOME=/usr/lib/jvm/java-8-oracle" >> /etc/environment
EOF

NGINX_CONTENT = ->(app_name, app_port) {
<<-EOF
upstream #{app_name}_server {
  server 127.0.0.1:#{app_port} fail_timeout=0;
}
server {
  listen 80;
  listen [::]:80 default ipv6only=on;
  server_name #{app_name}.ncr.systems www.#{app_name}.ncr.systems;
  access_log /var/log/nginx/#{app_name}.log combined;

  location / {
    proxy_set_header Host \\$host;
    proxy_set_header X-Real-IP \\$remote_addr;
    proxy_set_header X-Forwarded-For \\$proxy_add_x_forwarded_for;
    proxy_redirect off;

    if (!-f \\$request_filename) {
      proxy_pass http://#{app_name}_server;
      break;
    }
  }
}
EOF
}

NGINX_SETUP = ->(content) {
  <<-EOF
    apt-get install -y nginx
    echo "#{content}" > /etc/nginx/sites-available/app.conf
    cd /etc/nginx/sites-enabled
    rm -rf ./*
    ln -s ../sites-available/app.conf
    nginx -s reload
  EOF
}

ALL_SERVERS= {'nexus' => '172.20.20.11'}.freeze

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.provision "shell", privileged: true, inline: COMMON_SETUP

  config.vm.define "nexus" do |n1|
      n1.vm.hostname = "nexus"
      n1.vm.network "private_network", ip: ALL_SERVERS.fetch('nexus')
      n1.vm.provider("virtualbox") { |x| x.memory = 2*1024 }

      n1.vm.provision "shell", privileged: true, inline: JVM_SETUP
      n1.vm.provision "shell", privileged: true, inline: <<-EOF
        mkdir /opt/sonatype
        cd /opt/sonatype
        wget http://download.sonatype.com/nexus/3/latest-unix.tar.gz
        tar -zxvf latest-unix.tar.gz
        mv -v nexus-* nexus-release
        ln -s /opt/sonatype/nexus-release /usr/local/nexus
        echo "NEXUS_HOME=/usr/local/nexus" >> /etc/environment
        rm latest-unix.tar.gz
        chown -R vagrant /opt/sonatype
        chown -R vagrant /usr/local/nexus
      EOF
      n1.vm.provision "shell", privileged: false, inline: "/usr/local/nexus/bin/nexus start"
      n1.vm.provision "shell", privileged: true, inline: NGINX_SETUP.call(NGINX_CONTENT.call('nexus', 8081))
  end
end
