# Jupyter Hub Sandbox

This is a sandbox repo primarially for the purposes of documenting how to install Jupyter Hub
                 
## Installation and Configuration
                    
### OSX Install Dependancies
```
brew install python3
pip3 install --upgrade pip
pip3 install jupyterhub
pip3 install jupyter
npm install --global configurable-http-proxy
```

### Generate OpenSSL Certificates
```
mkdir -p ssl
openssl genrsa -out ssl/server.key 2048
openssl rsa -in ssl/server.key -out ssl/server.key
openssl req -sha256 -new -key ssl/server.key -out ssl/server.csr -subj '/CN=localhost'
openssl x509 -req -sha256 -days 365 -in ssl/server.csr -signkey ssl/server.key -out ssl/server.crt
```

## Jupyter
### Configuration
```
jupyter notebook --generate-config  # Writing default config to: ~/.jupyter/jupyter_notebook_config.py
```
~/.jupyter/jupyter_notebook_config.py
```
 79 c.NotebookApp.certfile = './ssl/server.crt'
 83 c.NotebookApp.client_ca = './ssl/server.key'
504 c.ContentsManager.root_dir = './jupyter/'
```
### Running
```
mkdir jupyter;
jupyter notebook # runs on http://localhost:8888

[I 22:06:00.846 NotebookApp] Serving notebooks from local directory: /Users/jamie/Dropbox/Programming/Sandbox/JupyterHub/jupyter
[I 22:06:00.846 NotebookApp] 0 active kernels 
[I 22:06:00.846 NotebookApp] The Jupyter Notebook is running at: https://localhost:8888/?token=72782a84fc628572dc8f437d5abe308658724bc7b13791a3
[I 22:06:00.846 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 22:06:00.847 NotebookApp] 
    
    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        https://localhost:8888/?token=72782a84fc628572dc8f437d5abe308658724bc7b13791a3
```


## Jupyter Hub 
### Configuration
```
mkdir jupyterhub;
jupyterhub --generate-config -f ./jupyterhub/jupyterhub_config.py
```

./jupyterhub/jupyterhub_config.py
```
101 c.JupyterHub.cookie_secret_file = 'jupyterhub/jupyterhub_cookie_secret'
111 c.JupyterHub.db_url = 'sqlite:///jupyterhub/jupyterhub.sqlite'
220 c.JupyterHub.ssl_cert = './ssl/server.crt'
225 c.JupyterHub.ssl_key = './ssl/server.key'
412 c.Spawner.notebook_dir = './jupyterhub/'
```

### Running
```
jupyterhub --config jupyterhub/jupyterhub_config.py 

[I 2017-04-04 22:07:28.524 JupyterHub app:724] Loading cookie_secret from /Users/jamie/Dropbox/Programming/Sandbox/JupyterHub/jupyterhub/jupyterhub_cookie_secret
[W 2017-04-04 22:07:28.556 JupyterHub app:365] 
    Generating CONFIGPROXY_AUTH_TOKEN. Restarting the Hub will require restarting the proxy.
    Set CONFIGPROXY_AUTH_TOKEN env or JupyterHub.proxy_auth_token config to avoid this message.
    
[W 2017-04-04 22:07:28.559 JupyterHub app:864] No admin users, admin interface will be unavailable.
[W 2017-04-04 22:07:28.559 JupyterHub app:865] Add any administrative users to `c.Authenticator.admin_users` in config.
[I 2017-04-04 22:07:28.559 JupyterHub app:892] Not using whitelist. Any authenticated user will be allowed.
[I 2017-04-04 22:07:28.579 JupyterHub app:1453] Hub API listening on http://127.0.0.1:8081/hub/
[I 2017-04-04 22:07:28.583 JupyterHub app:1176] Starting proxy @ http://*:8000/
22:07:28.792 - info: [ConfigProxy] Proxying https://*:8000 to http://127.0.0.1:8081
22:07:28.796 - info: [ConfigProxy] Proxy API at http://127.0.0.1:8001/api/routes
```

## Proxy

### Auth Tokens
```
export CONFIGPROXY_AUTH_TOKEN=`openssl rand -hex 32`
```
or
```
./jupyterhub/jupyterhub_config.py:     c.JupyterHub.proxy_auth_token = '0bc02bede919e99a26de1e2a7a5aadfaf6228de836ec39a05a6c6942831d8fe5'
~/.jupyter/jupyter_notebook_config.py  c.NotebookApp.token           = '0bc02bede919e99a26de1e2a7a5aadfaf6228de836ec39a05a6c6942831d8fe5'
```

### Configurable HTTP Proxy - verify
```
configurable-http-proxy --port 8001 --api-port 8888 --ssl-key ssl/server.key --ssl-cert ssl/server.crt 
```
    
# Urls
- https://localhost:8888/          - jupyter 
- https://localhost:8000/hub/login - jupyter hub https proxy  | login = system user/pass
- http://localhost:8001/hub/login  - jupyter hub              | login = system user/pass
- http://localhost:8001            - proxy api = Not Found 


## Filesystem Stucture
```
$ tree -a
├── README.md
├── ssl
│   ├── server.crt
│   ├── server.csr
│   └── server.key
└── workbooks
    ├── .ipynb_checkpoints
    │   └── Untitled-checkpoint.ipynb
    └── Untitled.ipynb
```
