# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/contrib-stretch64"
	config.vm.network "private_network", ip: "192.168.33.230"
  config.vm.hostname = "psbox.local"
  config.vm.synced_folder ".", "/var/www", :mount_options => ["dmode=777", "fmode=777"]

  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
    sudo unlink /etc/localtime
    sudo ln -s /usr/share/zoneinfo/Europe/Paris /etc/localtime
    
    sudo apt-get update
    
    echo "-- Installing debianconf --"
    sudo apt-get install -y debconf-utils
    
    echo "-- Installing dirmngr --"
    sudo apt-get install dirmngr
    
    echo "-- Installing unzip --"
    sudo apt-get install -y unzip
    
    echo "-- Installing aptitude --"
    sudo apt-get -y install aptitude
    
    echo "-- Updating package lists --"
    sudo aptitude update -y
    
    #echo "-- Updating system --"
    #//sudo aptitude safe-upgrade -y
    
    echo "-- Uncommenting alias for ll --"
    sed -i "s/#alias ll='.*'/alias ll='ls -al'/g" /home/vagrant/.bashrc
    
    echo "-- Installing curl --"
    sudo aptitude install -y curl
    
    echo "-- Installing apt-transport-https --"
    sudo aptitude install -y apt-transport-https
    
    echo "-- Adding GPG key for sury repo --"
    sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
    
    echo "-- Adding PHP 7 packages repo --"
    echo 'deb https://packages.sury.org/php/ stretch main' | sudo tee -a /etc/apt/sources.list
    
    echo "-- Updating package lists again after adding sury --"
    sudo aptitude update -y
    
    echo "-- Installing Apache --"
    sudo aptitude install -y apache2
    
    echo "-- Enabling mod rewrite --"
    sudo a2enmod rewrite
    
    echo "-- Configuring Apache --"
    sudo sed -i "s/AllowOverride None/AllowOverride All/g" /etc/apache2/apache2.conf
    
    echo "-- Adding MySQL GPG key --"
    wget -O /tmp/RPM-GPG-KEY-mysql https://repo.mysql.com/RPM-GPG-KEY-mysql
    sudo apt-key add /tmp/RPM-GPG-KEY-mysql
    
    echo "-- Adding MySQL repo --"
    echo "deb http://repo.mysql.com/apt/debian/ stretch mysql-5.7" | sudo tee /etc/apt/sources.list.d/mysql.list
    
    echo "-- Updating package lists after adding MySQL repo --"
    sudo aptitude update -y
    
    sudo debconf-set-selections <<< "mysql-community-server mysql-community-server/root-pass password $MYSQL_PASS"
    sudo debconf-set-selections <<< "mysql-community-server mysql-community-server/re-root-pass password $MYSQL_PASS"
    
    echo "-- Installing MySQL server --"
    sudo aptitude install -y mysql-server
    
    echo "-- Creating alias for quick access to the MySQL (just type: db) --"
    echo "alias db='mysql -u root -p$MYSQL_PASS'" >> /home/vagrant/.bashrc
    
    echo "-- Installing PHP stuff --"
    sudo aptitude install -y libapache2-mod-php7.3 php7.3 php7.3-pdo php7.3-mysql php7.3-mbstring php7.3-xml php7.3-intl php7.3-tokenizer php7.3-gd php7.3-imagick php7.3-curl php7.3-zip
    
    echo "-- Installing Xdebug --"
    sudo aptitude install -y php-xdebug
    
    echo "-- Installing libpng-dev (required for some node package) --"
    sudo aptitude install -y libpng-dev
    
    echo "-- Configuring xDebug (idekey = PHP_STORM) --"
    sudo echo "xdebug.remote_enable=1" >> /etc/php/7.3/mods-available/xdebug.ini
    sudo echo "xdebug.remote_connect_back=1" >> /etc/php/7.3/mods-available/xdebug.ini
    sudo echo "xdebug.remote_port=9001" >> /etc/php/7.3/mods-available/xdebug.ini
    sudo echo "xdebug.idekey=PHP_STORM" >> /etc/php/7.3/mods-available/xdebug.ini

    echo "-- Configuring Apache --"
    sudo rm /var/www/html -rf
    sudo sed -i s,/var/www/html,/var/www,g /etc/apache2/sites-available/000-default.conf
    
    echo "-- Restarting Apache --"
    sudo /etc/init.d/apache2 restart
    
    echo "-- Installing Git --"
    sudo aptitude install -y git
    
    echo "-- Installing Composer --"
    curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
    
    echo "-- Installing node.js -->"
    curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
    sudo aptitude install -y nodejs
    
    echo "-- Installing PHPEnv Tools--"
    sudo aptitude install -y ruby
    sudo aptitude install -y re2c
    sudo aptitude install -y apache2-dev
    sudo aptitude install -y libbz2-dev libcurl4-openssl-dev libjpeg-dev libmcrypt-dev libssl-dev libreadline-dev libtidy-dev libxslt1-dev libxml2-dev libxml++2.6-dev 
  SHELL
  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    ## PHPEnv
    echo "-- PHPEnv : Install --"
    curl -L http://git.io/phpenv-installer | bash
    ## PHPEnv : CLI
    echo "-- PHPEnv : .bashrc --"
    echo "" >> /home/vagrant/.bashrc
    echo 'export PHPENV_ROOT="/home/vagrant/.phpenv"' >> /home/vagrant/.bashrc
    echo 'if [ -d "${PHPENV_ROOT}" ]; then' >> /home/vagrant/.bashrc
    echo '  export PATH="${PHPENV_ROOT}/bin:${PATH}"' >> /home/vagrant/.bashrc
    echo '  eval "$(phpenv init -)"' >> /home/vagrant/.bashrc
    echo 'fi' >> /home/vagrant/.bashrc
    ## PHPEnv : Composer
    echo "-- PHPEnv : Composer --"
    git clone https://github.com/ngyuki/phpenv-composer.git $HOME/.phpenv/plugins/phpenv-composer
    ## PHPEnv : Apache
    echo "-- PHPEnv : Apache Version --"
    git clone https://github.com/garamon/phpenv-apache-version $HOME/.phpenv/plugins/phpenv-apache-version
    echo "-- PHPEnv : Apache Version Patch --"
    sed -i 's,\${PHPENV_APACHE_VERSION}",\${PHPENV_APACHE_VERSION}/libexec",g' $HOME/.phpenv/plugins/phpenv-apache-version/bin/rbenv-apache-version
    #sed -i 's,\${apxs_libexecdir}|,|,g' $HOME/.phpenv/plugins/php-build/bin/php-build
    #mkdir -p $HOME/.phpenv/plugins/php-build/share/php-build/before-install.d
    #curl -sSL https://raw.githubusercontent.com/ngyuki/php-build-apxs2/master/patch-apxs2.sh > $HOME/.phpenv/plugins/php-build/share/php-build/before-install.d/patch-apxs2.sh
    #chmod +x $HOME/.phpenv/plugins/php-build/share/php-build/before-install.d/patch-apxs2.sh
    echo "-- PHPEnv : Apache Version Definitions --"  
    echo -D --enable-fpm >> $HOME/.phpenv/plugins/php-build/share/php-build/default_configure_options
    echo --disable-fpm >> $HOME/.phpenv/plugins/php-build/share/php-build/default_configure_options
    echo --with-apxs2 >> $HOME/.phpenv/plugins/php-build/share/php-build/default_configure_options

    #for file in $HOME/.phpenv/plugins/php-build/share/php-build/definitions/*; do echo "configure_option -D --enable-fpm" >> $file; done
    #for file in $HOME/.phpenv/plugins/php-build/share/php-build/definitions/*; do echo "configure_option --disable-fpm" >> $file; done
    #for file in $HOME/.phpenv/plugins/php-build/share/php-build/definitions/*; do echo "configure_option --with-apxs2=/usr/bin/apxs" >> $file; done
  SHELL
end