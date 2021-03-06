---
title: 'AutoSploit v4.0'
date: 2019-10-26T13:49:00+01:00
draft: false
---

**Kang Asu**

[![](https://1.bp.blogspot.com/-hznXstvsNIU/XbWS02EpAxI/AAAAAAAABwE/pYEtH24crcAe_5emB7zF7Ktic5uhLTUJQCLcBGAsYHQ/s320/AutoSploit_1.jpeg)](https://1.bp.blogspot.com/-hznXstvsNIU/XbWS02EpAxI/AAAAAAAABwE/pYEtH24crcAe_5emB7zF7Ktic5uhLTUJQCLcBGAsYHQ/s1600/AutoSploit_1.jpeg)

**AutoSploit v4.0 - Automated Mass Exploiter**

As the name might suggest AutoSploit attempts to automate the [exploitation](https://www.kitploit.com/search/label/Exploitation "exploitation") of remote hosts. Targets can be collected automatically through Shodan, Censys or Zoomeye. But options to add your custom targets and host lists have been included as well. The available Metasploit modules have been selected to facilitate [Remote Code Execution](https://www.kitploit.com/search/label/Remote%20Code%20Execution "Remote Code Execution") and to attempt to gain Reverse TCP Shells and/or Meterpreter sessions. Workspace, local host and local port for MSF facilitated back connections are configured by filling out the dialog that comes up before the exploit component is started

[](https://www.blogger.com/u/1/null)  
_**Operational Security Consideration:**_  
Receiving back connections on your local machine might not be the best idea from an OPSEC standpoint. Instead consider running this tool from a VPS that has all the dependencies required, available.  
The new version of AutoSploit has a feature that allows you to set a proxy before you connect and a custom user-agent.  
  
**Installation**  
Installing AutoSploit is very simple, you can find the latest stable release [here](https://github.com/NullArray/AutoSploit/releases/latest "here"). You can also download the master branch as a [zip](https://github.com/NullArray/AutSploit/zipball/master "zip") or [tarball](https://github.com/NullArray/AutSploit/tarball/master "tarball") or follow one of the below methods;  
  
**Docker Compose**  
Using Docker Compose is by far the easiest way to get AutoSploit up and running without too much of a hassle.

```
git clone https://github.com/NullArray/AutoSploit.git  
cd Autosploit/Docker  
docker-compose run --rm autosploit
```

  
**Docker**  
Just using Docker.

```
git clone https://github.com/NullArray/AutoSploit.git  
cd Autosploit/Docker  
# If you wish to edit default postgres service details, edit database.yml. Should work out of the box  
# nano database.yml  
docker network create -d bridge haknet  
docker run --network haknet --name msfdb -e POSTGRES_PASSWORD=s3cr3t -d postgres  
docker build -t autosploit .  
docker run -it --network haknet -p 80:80 -p 443:443 -p 4444:4444 autosploit
```

Dev team contributor [Khast3x](https://github.com/khast3x "Khast3x") recently improved Docker operations as well as add more details to the README.md in the `Docker`subdirectory. For more information on deploying AutoSploit with Docker please be sure to click [here](https://github.com/NullArray/AutoSploit/tree/master/Docker "here")  
  
**Cloning**  
On any Linux system the following should work;

```
git clone https://github.com/NullArray/AutoSploit  
cd AutoSploit  
chmod +x install.sh  
./install.sh
```

AutoSploit is compatible with macOS, however, you have to be inside a virtual environment for it to run successfully. In order to accomplish this employ/perform the below operations via the terminal or in the form of a shell script.

```
sudo -s << '_EOF'  
pip2 install virtualenv --user  
git clone https://github.com/NullArray/AutoSploit.git  
virtualenv   
source /bin/activate  
cd   
pip2 install -r requirements.txt  
chmod +x install.sh  
./install.sh  
python autosploit.py  
_EOF
```

  
**Usage**  
Starting the program with `python autosploit.py` will open an AutoSploit terminal session. The options for which are as follows.

```
1. Usage And Legal  
2. Gather Hosts  
3. Custom Hosts  
4. Add Single Host  
5. View Gathered Hosts  
6. Exploit Gathered Hosts  
99. Quit
```

Choosing option `2` will prompt you for a platform specific search query. Enter `IIS` or `Apache` in example and choose a search engine. After doing so the collected hosts will be saved to be used in the `Exploit` component.  
As of version 2.0 AutoSploit can be started with a number of [command line](https://www.kitploit.com/search/label/Command%20Line "command line") arguments/flags as well. Type `python autosploit.py -h` to display all the options available to you. I've posted the options below as well for reference.

```
usage: python autosploit.py -[c|z|s|a] -[q] QUERY  
                            [-C] WORKSPACE LHOST LPORT [-e] [--whitewash] PATH  
                            [--ruby-exec] [--msf-path] PATH [-E] EXPLOIT-FILE-PATH  
                            [--rand-agent] [--proxy] PROTO://IP:PORT [-P] AGENT  
  
optional arguments:  
  -h, --help            show this help message and exit  
  
search engines:  
  possible search engines to use  
  
  -c, --censys          use censys.io as the search engine to gather hosts  
  -z, --zoomeye         use zoomeye.org as the search engine to gather hosts  
  -s, --shodan          use shodan.io as the search engine to gather hosts  
  -a, --all             search all available search engines to gather hosts  
  
requests:  
  arguments to edit your requests  
  
  --proxy PROTO://IP:PORT  
                        run behind a proxy while performing the searches  
  --random-agent        use a rando   m HTTP User-Agent header  
  -P USER-AGENT, --personal-agent USER-AGENT  
                        pass a personal User-Agent to use for HTTP requests  
  -q QUERY, --query QUERY  
                        pass your search query  
  
exploits:  
  arguments to edit your exploits  
  
  -E PATH, --exploit-file PATH  
                        provide a text file to convert into JSON and save for  
                        later use  
  -C WORKSPACE LHOST LPORT, --config WORKSPACE LHOST LPORT  
                        set the configuration for MSF (IE -C default 127.0.0.1  
                        8080)  
  -e, --exploit         start exploiting the already gathered hosts  
  
misc arguments:  
  arguments that don't fit anywhere else  
  
  --ruby-exec           if you need to run the Ruby executable with MSF use  
                        this  
  --msf-path MSF-PATH   pass the path to your framework if it is not in your  
                           ENV PATH  
  --whitelist PATH      only exploit hosts listed in the whitelist file
```

  
**Dependencies**  
_Note_: All dependencies should be installed using the above installation method, however, if you find they are not:  
AutoSploit depends on the following Python2.7 modules.

```
requests  
psutil
```

Should you find you do not have these installed get them with pip like so.

```
pip install requests psutil
```

or

```
pip install -r requirements.txt
```

Since the program invokes functionality from the [Metasploit Framework](https://www.kitploit.com/search/label/Metasploit%20Framework "Metasploit Framework") you need to have this installed also. Get it from Rapid7 by clicking [here](https://www.rapid7.com/products/metasploit/ "here").  
  
**Acknowledgements**  
Special thanks to [Ekultek](https://github.com/Ekultek "Ekultek") without whoms contributions to the project, the new version would have been a lot less spectacular.  
Thanks to [Khast3x](https://github.com/khast3x "Khast3x") for setting up Docker support.  
Last but certainly not least. Thanks to all who have submitted Pull Requests, bug reports, useful and productive contributions in general.  
  
**Active Development**  
If you would like to contribute to the development of this project please be sure to read [CONTRIBUTING.md](https://github.com/NullArray/AutoSploit/blob/master/CONTRIBUTING.md "CONTRIBUTING.md") as it contains our contribution guidelines.  
Please, also, be sure to read our [contribution standards](https://github.com/NullArray/AutoSploit/wiki/Development-information#contribution-standards "contribution standards") before sending pull requests  
If you need some help understanding the code, or want to chat with some other AutoSploit community members, feel free to join our [Discord server](https://discord.gg/DZe4zr2 "Discord server").  
  
**Note**  
If you happen to encounter a bug please feel free to [Open a Ticket](https://github.com/NullArray/AutoSploit/issues "Open a Ticket").  
Thanks in advance.  
  
**Translations**

*   [FR](https://github.com/NullArray/AutoSploit/blob/master/.github/.translations/README-fr.md "FR")
*   [ZH](https://github.com/NullArray/AutoSploit/blob/master/.github/.translations/README-zh.md "ZH")
*   [DE](https://github.com/NullArray/AutoSploit/blob/master/.github/.translations/README-de.md "DE")

  

**[Download AutoSploit](http://eunsetee.com/eJI1 "Download AutoSploit")**

**Regards**

**Kang Asu**