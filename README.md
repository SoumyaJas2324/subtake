# subtake

### Installation:
 ## git clone https://github.com/jakejarvis/subtake

***Requires Go.

>> go get github.com/jakejarvis/subtake

### Usage
   Options

    -f to-check.txt is the path to your list of subdomains to check. One subdomain per line. Required.
    -t is the number of threads to use. (Default: 10)
    -a skips CNAME check and sends requests to every URL. (Default: false, but Highly recommended.)
    -timeout is the number seconds to wait before timing out a check (Default: 10).
    -o results.txt is a filename to output results to. If the file ends with .json, subtake will automatically switch to JSON format.
    -v enables verbose mode. Displays all checks including not vulnerable URLs.
    -c Path to file containing JSON fingerprint configuration. (Default: ./fingerprints.json)
    -ssl enforces HTTPS requests which may return a different set of results and increase accuracy.
    
   ***This tool takes a list of CNAME records to check and outputs potential takeover candidates pointing to these services. But how in the world do we get a list of every CNAME on the internet?
   Conveniently, Rapid7 publishes a monthly list for us through their Project Sonar survey!
   
   ***Project Sonar is a security research project by Rapid7 that conducts internet-wide surveys across different services and protocols to gain insights into global exposure to common vulnerabilities. The data collected is available to the public in an effort to enable security research.
   
   ### One of their free monthly datasets is called Forward DNS
   
   ****where you’ll find .json files named xxxx-fdns_cname.json.gz. Within the repository, there's an automated script named sonar.sh, which downloads the dataset for you and outputs a simple text file of CNAMEs pointed to any of the services listed above. Once you've cloned the subtake repository and grabbed the timestamp part of the filename (the string that precedes -fdns_cname.json.gz)
   
   ### usage of the script is as follows:

>> ./sonar.sh 2019-03-30-1553989414 sonar_output.txt

***This new text file contains both active and abandoned subdomains pointing to any of the services listed above — we still need to narrow it down to the takeover candidates by attempting to actually resolve each of them, which is where subtake comes into play.

### For a detailed description of the different options you can play around with, see the full readme on GitHub — but here’s a simple example command that uses 50 threads to take the CNAMEs listed in sonar_output.txt and outputs potentially vulnerable subdomains to vulnerable.txt

>> subtake -f sonar_output.txt -c fingerprints.json -t 50 -ssl -a -o vulnerable.txt

****This could take quite a while — up to a day, depending on your CPU, memory, and bandwidth — so I usually run it on a VM in the cloud and use Linux’s screen command to keep it running and check in periodically. There will also be many unavoidable false positives that you'll need to check yourself by trying to claim the abandoned name on the corresponding service's portal, which is why I keep using the term potential takeovers.

## it's easy to use grep to narrow down your vulnerable.txt list even more:
>> grep -f grep.txt vulnerable.txt
    

### Resources

***sonar.sh can be used first to gather a list of CNAMEs collected by Rapid7/scan.io's Project Sonar. This list can then be passed into subtake to return subdomains not in use. sonar.sh is based off of scanio.sh.

***fingerprints.json can be modified to add or remove hosted platforms to probe for. Many obscure platforms are included, and removing fingerprints for services that are uninteresting to you can speed up the scan.

***If you plan on using a high number of threads to speed the process up, you may need to temporarily raise the ulimit of your shell:

ulimit -a          # show current limit (usually 1024)
ulimit -n 10000    # set waaaaay higher
ulimit -a          # check new limit



### After generating a list of all vulnerable subdomains, you can use my collection of domains invoked in bug bounty programs to narrow down valuable targets and possibly get some ca$h monie$$$.
**Examples

>> ./sonar.sh 2018-10-27-1540655191 sonar_all_cnames.txt

>> subtake -f sonar_all_cnames.txt -t 50 -ssl -a -o vulnerable.txt

### Subdomain Takeover Tips

    A great explanation of the risks of takeovers and steps to responsibly disclose takeovers to companies: https://0xpatrik.com/subdomain-takeover/
    
    A comprehensive list of what services are vulnerable (and the basis of fingerprints.json), 
    how to proceed once finding them: https://github.com/EdOverflow/can-i-take-over-xyz

***Services Checked

    Amazon S3
    Amazon CloudFront (no longer vulnerable?)
    Microsoft Azure
    Heroku
    GitHub Pages
    Fastly
    Pantheon.io
    Shopify
    Tumblr
    WordPress.com
    Ghost
    Surge
    Statuspage
    Bitbucket Pages
    UserVoice
    Zendesk
    Brightcove
    Big Cartel
    Acquia
    ReadMe.io
    MaxCDN
    Apigee
    Smugmug

To-Do

    Integrate sonar.sh into the main Go script as an option instead of input file.
    All-in-one Docker image to automatically download the latest FDNS Project Sonar file and check for takeover possibilities.
    Have sonar.sh pull domains to check for from fingerprints.json, instead of hard-coding them.

