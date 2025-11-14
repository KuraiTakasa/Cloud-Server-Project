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
![Launch Instance]()
![Launch Instance]()
![Launch Instance]()
![Launch Instance]()
![Launch Instance]()
![Launch Instance]()
