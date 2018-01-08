.. _Tutorial:

Appendix: Tutorial(Using macOS and VirtualBox)
------------------

Overview
~~~~~~~~

This is the procedure of "Getting Started" using macOS and VirtualBox.
You can execute Selenium with Python after you follow this procedure.

- You install CentOS in VirtualBox on macOS.
- You install Selenium and Firefox on CentOS.
- You write Python Script for Selenium.
- You run Selenium on CentOS.
- Firefox's window is run on macOS by XServer(XQuartz).

The required software is below:

- VirtualBox [#]_
- Vagrant [#]_
- XServer(XQuartz) [#]_

.. [#] https://www.virtualbox.org/
.. [#] https://www.vagrantup.com/
.. [#] https://www.xquartz.org/

Verified in the following environment:

- macOS High Sierra version 10.13.2
- Vagrant 2.0.1
- VirtualBox 5.1.30 r118389
- XQuartz 2.7.11

Installing CentOS
~~~~~~~~~~~~~~~~~

Host(macOS) :
::

  vagrant init centos/7
  mv Vagrantfile{,.bak}
  sed '/"private_network"/s/#//' Vagrantfile.bak > Vagrantfile
  vagrant up
  vagrant ssh

Guest(CentOS) :

Disable SELinux
::

  sudo sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
  exit

Host(macOS) :
::

  vagrant halt
  vagrant up
  vagrant ssh

Guest(CentOS) :
::

  sudo yum update -y

Installing Selenium
~~~~~~~~~~~~~~~~~~~

Guest(CentOS) :
::

  cd /tmp
  curl -O https://bootstrap.pypa.io/get-pip.py
  sudo python get-pip.py
  sudo pip install selenium

Downloading Selenium Driver
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Guest(CentOS) :
::

  curl -LO https://github.com/mozilla/geckodriver/releases/download/v0.17.0/geckodriver-v0.17.0-linux64.tar.gz
  tar xvf geckodriver-v0.17.0-linux64.tar.gz

Downloading Selenium Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Guest(CentOS) :
::

  curl -O http://selenium-release.storage.googleapis.com/3.6/selenium-server-standalone-3.6.0.jar

Installing Java & Firefox
~~~~~~~~~~~~~~~~~~~~~~~~~

Guest(CentOS) :
::

  sudo yum install -y java firefox

Running Selenium server
~~~~~~~~~~~~~~~~~~~~~~~

Guest(CentOS) :
::

  java -jar selenium-server-standalone-3.6.0.jar

Running XServer(XQuartz)
~~~~~~~~~~~~~~~~~~~~~~~~

Host(macOS) :

When you run XServer(XQuartz) on macOS, xterm is opened.
You execute the following command in xterm:

::

  xhost +

Writing a script
~~~~~~~~~~~~~~~~

Host(macOS) :

Open a new terminal and move to the same directory(`Installing CentOS`_).

::

  vagrant ssh

Guest(CentOS) :
::

  cd /tmp

  cat << 'EOF' > python_org_search.py
  from selenium import webdriver
  from selenium.webdriver.common.keys import Keys
  
  driver = webdriver.Firefox()
  driver.get("http://www.python.org")
  assert "Python" in driver.title
  elem = driver.find_element_by_name("q")
  elem.clear()
  elem.send_keys("pycon")
  elem.send_keys(Keys.RETURN)
  assert "No results found." not in driver.page_source
  driver.close()
  EOF

Running Selenium with Python
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Guest(CentOS) :
::

  addr=$(ip a | grep '192\.168' | sed 's/.*inet \([0-9.]*\).*/\1/' | sed 's/[0-9]*$/1/')
  export DISPLAY=${addr}:0.0
  PATH=$PATH:/tmp

  python python_org_search.py

Firefox will be run, open http://www.python.org and exit. If Firefox isn't run, there are some problems.
