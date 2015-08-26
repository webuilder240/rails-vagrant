Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.network :forwarded_port, guest: 3000, host: 3000
  config.vm.network :private_network, ip: "192.168.33.254"

  config.vm.synced_folder ".", "/vagrant"

  GUEST_RUBY_VERSION = '2.2.3'

  # 必要なパッケージをインストール
  config.vm.provision "shell", privileged: true, inline: <<-__SCRIPT__
    export DEBIAN_FRONTEND=noninteractive
    apt-get update -y

    # database
    apt-get install -y debconf-utils
    debconf-set-selections <<< "mysql-server-5.5 mysql-server/root_password password \"\""
    debconf-set-selections <<< "mysql-server-5.5 mysql-server/root_password_again password  \"\""
	apt-get install -y mariadb-server mariadb-client-5.5 libmariadbclient-dev
    apt-get install -y redis-server

	#cores 
	apt-get install -y htop wget vim tig

    # develop env
    apt-get install -y git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties libffi-dev

    # for testing
    apt-get install -y qt5-default libqt5webkit5-dev

	cd /vagrant
	chmod a+x dev-starter-script

	cd /etc/init.d/
	cp /vagrant/dev-starter-service dev-starter
	chmod a+x dev-starter

	apt-get install -y sysv-rc-conf
	sysv-rc-conf dev-starter on
  __SCRIPT__

  # RubyとNodeをコンパイル
  config.vm.provision "shell", privileged: false, inline: <<-__SCRIPT__
    # install rbenv
    git clone git://github.com/sstephenson/rbenv.git ~/.rbenv
    git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
    git clone git://github.com/sstephenson/rbenv-gem-rehash.git ~/.rbenv/plugins/rbenv-gem-rehash
    echo $'export PATH="\$HOME/.rbenv/bin:\$HOME/.rbenv/plugins/ruby-build/bin:\$PATH"' >> ~/.bashrc
    echo $'eval "\$(rbenv init -)"' >> ~/.bashrc
    echo $'gem: --no-ri --no-rdoc' > ~/.gemrc

    export PATH="$HOME/.rbenv/bin:$HOME/.rbenv/plugins/ruby-build/bin:$PATH"
    eval "$(rbenv init -)"

    #{defined?(GUEST_RUBY_VERSION) ? "rbenv install %s; rbenv global %s; gem install bundler" % [GUEST_RUBY_VERSION, GUEST_RUBY_VERSION] : ''}
  __SCRIPT__

  config.vm.provider "virtualbox" do |vb|
    vb.memory = ENV["VM_MEMORY"] || "2048"
    vb.customize ["guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold", 10000]
  end
end
