# Autobench

* This is a Work in Progress *

Custom built tool for provisioning a server in a bare metal automation system (MAAS in this instance),
and kicking off a job of your choice to run and push to a Phoromatic server automatically.

## Automated Server Provisioning to Phoromatic
Revisit, existing code was working with Ubuntu MAAS, need to circle back and clean it up
prov role was using for spinning up resources in maas and injecting the benchmark file

### Manual Client Provisioning and Push to Phoromatic

The client tooling is useful for running a test on a client and pushing it to phoromatic
automatically.

## Setting up Phoromatic Server on Ubuntu 20.04 LTS

Uses Ubuntu 20.04LTS for the moment, can be expanded to other OS. You can run the server
on metal or VM:

```
apt update
apt dist-upgrade
apt install -y php-cli php-zip php-xml php-sqlite3 php-gd
git clone https://github.com/phoronix-test-suite/phoronix-test-suite.git
cd phoronix-test-suite/
./install-sh
```

Can then start the server in tmux:
```
phoronix-test-suite start-phoromatic-server
```

Modify /etc/phoronix-test-suite.xml and set desired remote access ports:
```
    <Server>
      <RemoteAccessPort>5001</RemoteAccessPort>
      <Password></Password>
      <WebSocketPort>5000</WebSocketPort>
      <AdvertiseServiceZeroConf>TRUE</AdvertiseServiceZeroConf>
      <AdvertiseServiceOpenBenchmarkRelay>TRUE</AdvertiseServiceOpenBenchmarkRelay>
      <PhoromaticStorage>~/.phoronix-test-suite/phoromatic/</PhoromaticStorage>
    </Server>
```

- Connect to http://ip:remoteaccesport in browser and set up initial user
- Once logged in, under systems, obtain phoronix url: ex: 10.0.100.178:5001/HNKOOO


## Client Setup

Modify `env_vars.yml` and set `phoromatic_endpoint` to the url you obtained above. This will tell the client where to upload its results.

Run:

```
apt install -y ansible
ansible-playbook -i localhost bench.yml
```

This will install the necessary packages, setup the client and start the benchmark automatically. You can attach to the session with a `tmux attach -d` from the client to view the run. You will also need to click on systems in Phoromatic and approve the new machine that pops up.

## Creating a custom suite

Under Phoromatic Web Site, go to Tests, Build test suite. From there, give the test suite a name and assemble the desired tests you want in the suite. Once created, it will create a suite-definition on the phoromatic server. We are going to want to copy it from there and add it to our Ansible so that we can redeploy it on many hosts.

```
/var/lib/phoronix-test-suite/phoromatic/accounts/HNKOOO/suites/testtest-1.0.0/suite-definition.xml
```

You will want to copy the contents of that file to `roles/bench/files/hw-qual-1.0.0/suite-definition.xml` or you can the existing one as a start.

Modify `env_vars.yml` to reflect that updated test: `local/hw-qual-1.0.0` (pts/compress-gzip was the original example given)

If you run `ansible-playbook -i localhost bench.yml` again on the client, it should update things and kick off a new test. Ultimately it creates `start_benchmarks.sh` in the /root of the client machine, so you can edit and adjust that file as needed to rerun tests instead of running the ansible.

## Results
Once results are successfully uploaded, you can load Phoromatic click the results at the top, and select the various runs and compare them together. It may take a few seconds for the results to load after hitting compare.

If you have issues downloading PDFs, you may need to increase the php `max_execution_time` from 30 to 360 by editing `/etc/php/7.4/cli/php.ini` on the Phoromatic server side. You can view the logs in `/var/log/phoromatic.log` if you have problems.
