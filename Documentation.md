# Documentation #
This document contains step-by-step instructions on how I created my cloud server project.

### Step 1 ###
First, we launch an instance.

![Launch Instance](https://github.com/user-attachments/assets/db75539d-9875-4259-b102-f992d4fc73ed)

Name it.

![Launch Instance](https://github.com/user-attachments/assets/33730246-833c-46a2-9823-987ceeb9c6a0)

Then select the Ubuntu operating system.

![Launch Instance](https://github.com/user-attachments/assets/7d2dc94a-07c6-4918-80f7-e98ab7ba0c8b)

Select an instance type.

![Launch Instance](https://github.com/user-attachments/assets/59b800f3-232a-447b-84b8-f0696eed4956)

Now select an existing key pair, or create a new one.

![Launch Instance](https://github.com/user-attachments/assets/9bc4f1ce-2261-477f-8b4e-932fa7f563ff)

Create a new security group so we can edit it later. Additionally, allowing access from anywhere is risky; however, my main network IP address is dynamic, and I would need to update the allowed IP address in my network settings each time I log in.

![Launch Instance](https://github.com/user-attachments/assets/3a221313-5ef0-48c7-9f0a-2b7a8d1296df)

Leave the storage as the default configuration.

![Launch Instance](https://github.com/user-attachments/assets/9333b393-0fa8-430d-91a3-c0e1cba14687)

Now we simply launch.

![Launch Instance](https://github.com/user-attachments/assets/acbd33a7-355e-4660-b216-aec82102637a)

You should see your instance running.

![Launch Instance](https://github.com/user-attachments/assets/ee471da8-8695-4258-865b-bd2e82f9d780)


### Step 2 ###
Now we select our instance and click connect.

![Launch Instance](https://github.com/user-attachments/assets/65a8091e-30bc-4aed-8780-ddaf9d45619c)

Connect using the public IP address option.

![Launch Instance](https://github.com/user-attachments/assets/d8cd8dae-4c9b-4baa-814e-9aec6496ae45)

Once in the terminal, run this. This installs Apache.

```
sudo apt update
sudo apt install apache2 -y
```

Now run this to enable and start Apache.

```
sudo systemctl enable apache2
sudo systemctl start apache2
```

Now we have to allow HTTP traffic to our server. Return to the EC2 dashboard and click the Security tab. Click on the security group that you created.

![Launch Instance](https://github.com/user-attachments/assets/a08301a9-c0d0-4876-ba3a-67611d309397)

When you see this interface, navigate to the inbound rules tab and click the 'edit inbound rules' button.

![Launch Instance](https://github.com/user-attachments/assets/caf7327b-eda4-467f-abd8-b46a7b517ae1)

Click 'Add rule'.

![Launch Instance](https://github.com/user-attachments/assets/e3418a65-a8b0-4974-a8ea-615e0dcd2e8a)

Add the following rule and hit save.

![Launch Instance](https://github.com/user-attachments/assets/56c6d7e7-346d-434c-8fd7-8adf164750f7)

Now, when you search your server's public IP, you will see this.

![Launch Instance](https://github.com/user-attachments/assets/e229a7a5-6f94-4518-90bb-683b36c2c797)


### Step 3 ###
Now we install WireGuard.

```
sudo apt install wireguard -y
```

We need to enable IP forwarding. Run this first. Then, uncomment the IP forward line.

```
sudo nano /etc/sysctl.conf
```

![Launch Instance](https://github.com/user-attachments/assets/a6e5d6c0-de72-4e36-b394-44a4c0618512)

Save the file and run this command.

![Launch Instance](https://github.com/user-attachments/assets/9f27dbc1-61a1-4e6c-9856-f31cc4ca4bed)

Now run the following command to generate a key pair for the server.

![Launch Instance](https://github.com/user-attachments/assets/60f2d4b2-a31e-4d09-92c3-bcb77bdb3502)

Now we take the private key we just generated and create a WireGuard config file. First, run this and copy the key. Run the second command and paste the text in the picture, replacing the private key with yours.

```
sudo cat /etc/wireguard/server_private.key
```
```
sudo nano /etc/wireguard/wg0.conf
```

![Launch Instance](https://github.com/user-attachments/assets/d5325669-632b-437f-8973-0bb14fec73d9)

Now run these 2 commands to start WireGuard.

```
sudo systemctl start wg-quick@wg0
sudo systemctl enable wg-quick@wg0
```

![Launch Instance](https://github.com/user-attachments/assets/01116a76-c7fb-4086-9737-3a781f7d3f03)

Lastly, we add another inbound rule in our security group.

![Launch Instance](https://github.com/user-attachments/assets/57c6b0ad-56a1-4726-85af-109d493d111d)


### Step 4 ###
Now we create 2 scripts.

```
cd /home/ubuntu/
mkdir vpn-portal
```

Then run this command and paste the following script. This script runs a simple web server that provides a user interface for your Bash script. It serves a webpage with a form where you can enter a username. When you submit the form, this app executes the create_user.sh script with that username and then displays a download link for the newly created .conf file.

```
nano app.py
```

```
from flask import Flask, render_template, request, send_from_directory
import subprocess

app = Flask(__name__)

# Folder where .conf files are stored
CONFIG_FOLDER = "/var/www/html"

@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        username = request.form.get('username')

        if not username:
            return render_template('index.html', message="Enter a username.")

        # Call the create_user.sh script
        result = subprocess.run(
            ["/home/ubuntu/vpn-portal/create_user.sh", username],
            capture_output=True,
            text=True
        )

        if result.returncode != 0:
            return render_template('index.html', message="Error creating user.")

        # Extract ONLY the last line (filename)
        conf_file = result.stdout.strip().split("\n")[-1]

        return render_template('index.html', download=conf_file)

    return render_template('index.html')


@app.route('/download/<filename>')
def download_file(filename):
    return send_from_directory(CONFIG_FOLDER, filename, as_attachment=True)


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
```

Now, type this command and paste the following script. This script automates the process of adding a new user to your WireGuard VPN. It takes a username as an argument, generates a new private/public key pair for them, adds their public key as a new [Peer] to the main wg0.conf server file, and then creates a separate client config file (e.g., user.conf) in the /var/www/html directory for the user to download.

```
nano create_user.sh
```

```
#!/bin/bash

# Username passed from Flask
USERNAME=$1

# Directories
WG_DIR="/etc/wireguard"
WG_CONF="$WG_DIR/wg0.conf"
WWW_DIR="/var/www/html"

# Your actual EC2 public IP
ENDPOINT_IP="13.127.91.166"

# Generate private and public keys
wg genkey | tee /tmp/${USERNAME}_privatekey | wg pubkey > /tmp/${USERNAME}_publickey
PRIVATE_KEY=$(cat /tmp/${USERNAME}_privatekey)
PUBLIC_KEY=$(cat /tmp/${USERNAME}_publickey)

# Generate a random client IP in the 10.0.0.x range
IP="10.0.0.$((100 + RANDOM % 100))"

# Add peer to server config if not already present
if ! grep -q "$PUBLIC_KEY" $WG_CONF; then
cat <<EOF | sudo tee -a $WG_CONF

[Peer]
# $USERNAME
PublicKey = $PUBLIC_KEY
AllowedIPs = $IP/32
EOF
fi

# Create the client config file
cat <<EOF > $WWW_DIR/${USERNAME}.conf
[Interface]
PrivateKey = $PRIVATE_KEY
Address = $IP/32
DNS = 1.1.1.1

[Peer]
PublicKey = $(sudo wg show wg0 public-key)
Endpoint = ${ENDPOINT_IP}:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
EOF

# Apply updated WireGuard configuration immediately
sudo wg addconf wg0 <(wg-quick strip wg0)

# Output ONLY the config filename so Flask can detect it
echo "${USERNAME}.conf"
```

Now we make a simple HTML webpage.

```
mkdir template
nano index.html
```

```
<!DOCTYPE html>
<html>
<head>
  <title>WireGuard VPN Portal</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 40px; background: #fafafa; }
    input, button { padding: 8px; margin: 6px; }
  </style>
</head>
<body>
  <h2>Connect to Aamir's VPN</h2>
  <form method="post">
    <input type="text" name="username" placeholder="Username" required>
    <input type="password" name="password" placeholder="Password (optional)">
    <button type="submit">Generate VPN Config</button>
  </form>
  {% if message %}
    <p style="color:red;">{{ message }}</p>
  {% endif %}
  {% if download %}
    <p>Your config file is ready: <a href="/download/{{ download }}">Download {{ download }}</a></p>
  {% endif %}
</body>
</html>
```

We need to ensure that Python is installed and Apache is temporarily disabled.

```
sudo apt install python3-pip -y
sudo apt install python3-flask -y
sudo systemctl stop apache2
sudo systemctl disable apache2
```

Now, for the website to work, you need to run the .py file through Python.

```
sudo python3 /home/ubuntu/vpn-portal/app.py
```
![Launch Instance]()
![Launch Instance]()
![Launch Instance]()
![Launch Instance]()
![Launch Instance]()
![Launch Instance]()
![Launch Instance]()
![Launch Instance]()
![Launch Instance]()
![Launch Instance]()
