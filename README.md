# CST8915 Lab 2: Algonquin Pet Store Refactored

**Student Name**: Mimi Dib
**Student ID**: 040829779  
**Course**: CST8915 Full-stack Cloud-native Development
**Semester**: Winter 2026

---

## Demo Video

🎥 [Watch Demo Video](https://youtu.be/uOHw9_9Wa_g)

## Links

- [order-service Repository](https://github.com/mimidib/order-service-lab2-demo)
- [product-service Repository](https://github.com/mimidib/product-service-lab2)
- [store-front Repository](https://github.com/mimidib/store-front-lab2)

---

## Notes & Reflection
Prof Ramy, please note that durign my video the VM was set to auto-shutdown to prevent racking up costs and it interrupted my vm demo but I have provided all of the components required.

### What changes did you make to the order-service and product-service to comply with the Configurations and Backing Services factors of the 12-Factor App methodology?
- **order-service changes:** We pasted the index.json from the lab 2 repo order-service folder to our current one where it replaced the hard-codeed `const TABBITMQ_URL = 'amqp://localhost';` with the environment variable `const RABBITMQ_CONNECTION_STRING = process.env.RABBITMQ_CONNECTION_STRING;` which also included a fallback to localhost incase not defined (Which i dont believe we would use in prod). Everywhere we used the original `RABBITMQ_URL` const, we replaced it with the connection string referencing the environment variable. We also added the .env file and installed `dotenv` dependency which was added to the `package.json` file

- **product-service changes:** We replaced the main.rs file with the Lab2 provided file which references the `.env` variables instead of hard coded values via `dotenv().ok()`, and we grab the port from the `.env` file using `let port: u16 = env::var("PORT")`. 

Both of these changes comply with the factors Factor 3: Configurations factor to store config in the environment, as well as the Factor 4: Backing services factor to treat backing services as attaches resources, so that we can easily swap out these services without modifying application code, and only need to change environment variable values. Fro example, now with these changes , if I wanted to host product-service on another VM, all I have to do is change the connection url store-front uses, replacing with the appropriate public ip and port!

### Why is it important to use environment variables instead of hard-coding configurations in your application?
When we use hard-coded values it makes any resource changes complex and isn't good practice as it makes changes prone to mistakes. With Backing services factor implemented it allows to quick and smooth resource swaps without changing application code, changing the environment variables addressable URL.

### Why is it important to have separate repositories for each microservice? How does this help maintain independence and scalability of each service?

Complying to the Factor 1: Code Base factor
- Ensuring one codebase per app is tracked in revision control with each code base allowing for a 1:N relationship for deploys is beneficial because it isolates the entire application improving independence by isolating changes and failures to one application improving troubleshooting, improves resiliency to not shut down the entire app, and allows for more focussed application code to satisfy business requirements.
Furthermore, having each application in its own repository allows for better scalability as the code can be bundled and deployed separately into application specific containers, allowing us to scale the applicaitons that have higher demand and optimize resources and cost in the process. 

Separate repositories are crucial as they enforce the **Code Base** principle, ensuring each service is fully decoupled for independent deployment lifecycles. This prevents a failure in one service from halting others, protecting uptime and SLOs. For scalability, each service can have custom elasticity configurations, allowing cost-effective scaling only where needed. From an SRE perspective, this isolation confines incidents, making troubleshooting easier with targeted telemetry and clear runbooks, which significantly reduces MTTR.


# Additional Notes & Issues

**Setup:**
- order-service-lab2-demo repo 
    - VM: vm0-cst8915-lab2
    - Public ip: 20.104.102.185
- RabbitMQ repo: 26W_CST8915-Lab1 (Ramy's repo) 
    - VM: vm2-2-cst8915-lab2
    - Public ip: 20.220.219.133
- product-service-lab2 repo
    - VM: vm1-product-service
    - Public ip: 20.151.107.149
- store-front-lab2 repo
    - VM: vm0-cst8915-lab2 (ran out of azure VM quota)
    - Public ip: 20.220.219.133

**RabbitMQ VM Setup**
1. Create VM & add the inbound port range for rabbitMQ running on 15672 (Any protocol) (done)
2. SSH to VM using VSCode `ssh -i "C:\Users\Mimi\.ssh\mimi-key1.pem" mimi@20.220.218.231`
**Remote SSH issue** the issue I ran into was connecting ANY other remote server via Remote SSH in VSCode. I spent 1h+ troubleshooting to find its likely the Azure CLI isn't found or is corrupted. I found this out via while attempting to connect to the remote host, opening the output via CTRL+` hitting 'output' and selecting the task 'Remote SSH' to find the following logs in the image provided below. Please note the configuration file with the setup is correct for all, confirming the path and public IPs, although the only remote ssh that works is 20.104.102.185.
![Remote SSH Error Logs & Config File](/CST8915-lab2-petstore-12-factor/CST8915-lab2-petstore-12-factor/img/ssh-issue.png)  

The only way I was able to fix it even after trying to use Bastion and failing was ensuring to take the ssh command from Azure portal and pasting my .pem file path in the provided input and copying that > pasting it in the config file and only connecting that way. Although the commands were the same, and the process was the same on vscode, the only difference was I got the command from Azure portal and that worked- it might be I had an extra space in the command begining or trail that I couldn't see.  
Command that wasn't accepted: `ssh -i "C:\Users\Mimi\.ssh\mimi-key1.pem" mimi@20.220.218.231`  
Command that worked: `ssh -i "C:\Users\Mimi\.ssh\mimi-key1.pem" mimi@20.220.218.231` Do you see a difference? I don't but for some reason it didn't accept the first one.
![Remote SSH Fix](/CST8915-lab2-petstore-12-factor/img/ssh-fix.png)
![SSH Config after fix](/CST8915-lab2-petstore-12-factor/img/ssh-config-fixed.png)

Before I could even clone the repository, going into the folders and creating a `Lab2` folder to clone it in, caused me to disconnect from RemoteSSH: 
![Disconnecting from Remote SSH due to creating Folder](/CST8915-lab2-petstore-12-factor/img/reconnecting.png)
3. Clone 26W_CST8915-Lab1 & run RabbitMQ from its dedicated VM:
    1. cd into `RabbitMQ/` & run `sudo ./rabbitmq-quick-install-script-Ubuntu_24_04.sh`
    2. `sudo systemctl start rabbitmq-server`
    3. Optionally check status: `sudo systemctl status rabbitmq-server`
    4. add new user and set permissions as per Lab instructions
    5. to access the management ui i would need to go to the new vm public ip (on  the bigger vm 20.220.219.133:15672 and login with username: guest and pass: guest)
    6. restart rabbitmq `sudo systemctl restart rabbitmq-server`
    ![RabbitMQ on VM](/CST8915-lab2-petstore-12-factor/img/rabbit-mq-on-vm.png)


**Store-front VM Setup (sharing with RabbitMQ VM due to quota limitations)**
1. Create VM & add the inbound port range for store-front running on 8080 (done)
2. SSH to VM using VSCode (`ssh -i "C:\Users\Mimi\.ssh\mimi-key1.pem" mimi@20.220.218.231`)
3. Clone store-front-lab2 & run store-front from its dedicated VM:
    Once connecting I attempted to clone the repository but ran into permission denied issues- I believe the issue is my vm doesn't have a SSH key to allow for git clone on this VM- but truthfully I'm not sure why that's needed when the repository is public. I created a new one with ssh-keygen and added it to my Github Account SSH keys.
    it was at this point that i realized the size of my Vm's- although they were 3 and the Lab required 4, the sizes are too small to actually use it to demonstrate I understand. I need to create more B2ls_v2 sixzes because this doidn't keep making me reconnect. I here decided to delete product-service vm shown in this image and make another one with more ram in the size B2als (has 4 ram rather than 1)
    1. Install npm and install dependencies (`sudo apt install npm`, `npm install @vue/cli-service@latest`, `npm install`) & run with `npm run serve`
    2. visit in ui with `http://20.220.219.133:8080`
    3. 


**product-service VM Setup**
1. git clone 
2. install dependencies 
    - `sudo apt update`
    - `sudo apt install build-essential`
    - `curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh`
    - `source "$HOME/.cargo/env"`
    - run with `cargo run`

    Product service running in B2als VM:
    ![running](/CST8915-lab2-petstore-12-factor/img/running.png)

**order-service Vm Setup**
1. This is the VM I initially created from the Lab1 exercise. I already have everything set up but for the sake of consistency:
Dependencies:
- `sudo apt update`
- `curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -`
- `sudo apt update`
- `sudo apt install -y nodejs`
- `npm install`
- run with `node index.js`

Order service running in vm:
![running order-serivce](/CST8915-lab2-petstore-12-factor/img/order-service-running.png)