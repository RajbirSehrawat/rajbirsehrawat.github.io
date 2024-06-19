# How to deploy a FastAPI app to AWS EC2 server

'''
pip install fastapi uvicorn gunicorn psycopg2-binary
'''


'''
sudo nano /etc/systemd/system/gunicorn.socket
''''

Add the following code to it:


'''
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
'''


Save and close that file. Next we create a service file:


'''
sudo nano /etc/systemd/system/gunicorn.service
'''


Add the following code to it:


'''
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/apiv1
ExecStart=/home/ubuntu/apiv1/env/bin/gunicorn \
          --access-logfile - \
          --workers 5 \
          --bind unix:/run/gunicorn.sock \
          --worker-class uvicorn.workers.UvicornWorker \
          app:app

[Install]
WantedBy=multi-user.target
'''

Save and close as well.
Next start the Gunicorn socket:


'''
sudo systemctl start gunicorn.socket
'''

Then enable it by:

'''
sudo systemctl enable gunicorn.socket
'''


Now that Gunicorn is set up, next we’ll configure Nginx to pass traffic to the process.
Start by creating and opening a new server block in Nginx’s sites-available directory:


'''
sudo nano /etc/nginx/sites-enabled/api
'''

Add the following lines to it:

'''
server {
    listen 80;
    server_name server_domain_or_IP;
    location / {
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
'''

Save this file as well. Test that the config is okay by:

If all is well then we need to restart the services so go aheda and:


'''
sudo nginx -t
sudo systemctl daemon-reload
sudo systemctl restart gunicorn
sudo systemctl restart nginx
'''


One last step is to ensure that you have allowed HTTP access via port 80 on your instance's security group section. Do the same for port 443 if your app is served over HTTPS.
If all the above is done then go to your instance's public IP address or domain and hopefully you will see your app being displayed there.
