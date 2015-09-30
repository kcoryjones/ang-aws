# -*- mode: ruby -*-
# vi: set ft=ruby :

# set up our provisioning scripts as ruby variables
$once = <<ONCE
    # update and upgrade
    apt-get update
    apt-get upgrade -y

    # install java runtime environment and dynamodb local
    javascript shell accessible at http://192.168.7.24:8000/shell/
    apt-get install default-jre -y
    mkdir /dynamodb
    wget http://dynamodb-local.s3-website-us-west-2.amazonaws.com/dynamodb_local_latest.tar.gz -O /dynamodb/dynamodb.tar.gz
    tar xvzf /dynamodb/dynamodb.tar.gz --directory /dynamodb
    rm /dynamodb/dynamodb.tar.gz
    java -Djava.library.path=./DynamoDBLocal_lib -jar /dynamodb/DynamoDBLocal.jar -sharedDb -cors * &

    # install aws cli latest version
    curl -O https://bootstrap.pypa.io/get-pip.py
    python get-pip.py
    pip install awscli
    rm get-pip.py

    # install zip and git
    apt-get install -y zip git

    # install nodejs and npm - the curl line here lets us install latest version
    curl -sL https://deb.nodesource.com/setup_dev | sudo bash -
    apt-get install -y nodejs

    # npm install global packages for structure and building
    npm install -g aws-sdk bcrypt-nodejs grunt-cli yo bower
    npm install -g git+https://github.com/kcoryjones/generator-ang-aws.git

    # install nginx
    apt-get install -y nginx

    # nginx tweaks - sendfile off, remove default server block, add our server block
    sed -i "s/sendfile on/sendfile off/g" /etc/nginx/nginx.conf
    rm /etc/nginx/sites-enabled/default
    ln -s /var/www/site /etc/nginx/sites-enabled/
ONCE

$always = <<ALWAYS
    # aws from command line
    alias aws="/usr/local/bin/aws"

    # start dynamodb local
    java -Djava.library.path=./DynamoDBLocal_lib -jar /dynamodb/DynamoDBLocal.jar -sharedDb -cors * &

    # latest npm dependencies
    test -d /var/www/app && npm install --prefix /var/www/app --no-bin-links
    test -d /var/www/lambda && npm install --prefix /var/www/lambda/login --no-bin-links
    test -d /var/www/lambda && npm install --prefix /var/www/lambda/api --no-bin-links

    # aws credentials and configuration
    test -d /home/vagrant/.aws && echo '/home/vagrant/.aws exists' || mkdir /home/vagrant/.aws
    test -f /var/www/aws_credentials && ln -s /var/www/aws_credentials /home/vagrant/.aws/credentials || echo 'no aws credential file'
    test -f /var/www/aws_config && ln -s /var/www/aws_config /home/vagrant/.aws/config || echo 'no aws config file'
    chown -R vagrant:vagrant /home/vagrant/.aws

    # restart nginx
    service nginx restart
ALWAYS

Vagrant.configure(2) do |config|
    config.vm.box = "ubuntu/trusty64"
    config.vm.network "private_network", ip: "192.168.7.24"
    config.vm.synced_folder ".", "/var/www"
    config.vm.provider "virtualbox" do |vb|
        vb.memory = "512"
        vb.name = "ang-aws"
    end
    config.vm.provision "once", type: "shell", inline: $once
    config.vm.provision "always", type: "shell", inline: $always, run: "always"
end