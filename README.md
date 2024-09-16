# OrionSdk
This project contains a python client for interacting with the SolarWinds Orion API

# API Documentation
For documentation about the SolarWinds Orion API, please see the wiki, tools, and sample code (in languages other than Python) in the main OrionSDK project.

# Install
pip install orionsdk

# Usage
import orionsdk

swis = orionsdk.SwisClient("server", "username", "password")

aliases = swis.invoke('Metadata.Entity', 'GetAliases', 'SELECT B.Caption FROM Orion.Nodes B')

print(aliases)

# SSL Certificate Verification
Initial support for SSL certificate valuation was added in 0.0.4. To enable this, you will need to save the self-signed cert to a file. One way of doing this is with OpenSSL:

openssl s_client -connect server:17778
Then add an entry to your hosts file for SolarWinds-Orion and you will be able to verify via doing the following:

import orionsdk
swis = orionsdk.SwisClient("SolarWinds-Orion", "username", "password", verify="server.pem")
swis.query("SELECT NodeID from Orion.Nodes")

# Setting Timeout
import orionsdk
import requests
session = requests.Session()
session.timeout = 30 # Set your timeout in seconds
swis = orionsdk.SwisClient("SolarWinds-Orion", "username", "password", verify="server.pem", session=session)
swis.query("SELECT NodeID from Orion.Nodes")


# Setting Retry
import orionsdk
import requests
from requests.adapters import HTTPAdapter
from requests.packages.urllib3.util.retry import Retry


def retry_session(retries=3,
                  backoff_factor=0.3,
                  status_forcelist=(500, 502, 504)):
    session = requests.Session()
    retry = Retry(
        total=retries,
        read=retries,
        connect=retries,
        backoff_factor=backoff_factor,
        status_forcelist=status_forcelist)
    adapter = HTTPAdapter(max_retries=retry)
    session.mount('http://', adapter)
    session.mount('https://', adapter)
    return session


swis = orionsdk.SwisClient(
    "SolarWinds-Orion",
    "username",
    "password",
    verify="server.pem",
    session=retry_session())
swis.query("SELECT NodeID from Orion.Nodes")
