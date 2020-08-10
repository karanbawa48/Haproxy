 config.vm.define "test" do |test|
  test.vm.box = "bento/ubuntu-18.04"
  test.vm.network "private_network", ip: "50.1.1.7"
  test.vm.network :forwarded_port, guest: 10, host: 1010
  end
  config.vm.define "test1" do |test1|  
  test1.vm.box =  "bento/ubuntu-18.04"
test1.vm.network "private_network", ip: "50.1.1.8"  
test1.vm.network :forwarded_port, guest: 40, host: 3306
config.vm.provider "virtualbox" do |test1|
  test1.memory = 500
  test1.cpus = 2  
end
end
end
