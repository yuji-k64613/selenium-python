.. _Tutorial:

Appendix: Tutorial
------------------

Overview
~~~~~~~~

This is the procedure of "Getting Started" using Mac and VirtualBox.
If you execute it according to the procedure, "Selenium with Python" will execute.

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

Disable selinux
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

Runing Selenium server
~~~~~~~~~~~~~~~~~~~~~~

Guest(CentOS) :
::

  java -jar selenium-server-standalone-3.6.0.jar

Runing XServer(XQuartz)
~~~~~~~~~~~~~~~~~~~~~~~

Host(macOS) :

When you start up the XServer(XQuartz), the terminal open,
You type the following command at the terminal:

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

Runing Selenium with Python
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Guest(CentOS) :
::

  addr=$(ip a | grep '192\.168' | sed 's/.*inet \([0-9.]*\).*/\1/' | sed 's/[0-9]*$/1/')
  export DISPLAY=${addr}:0.0
  PATH=$PATH:/tmp

  python python_org_search.py

After firefox starts up, "http://www.python.org" will be opened.
After that, if firefox finished, it works normally.

