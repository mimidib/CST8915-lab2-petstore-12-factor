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
Prof. Ramy—please note that during my video the VM was set to auto-shutdown to prevent racking up costs; it interrupted my VM demo but I have provided all required components.

### What changes did you make to the order-service and product-service to comply with the Configurations and Backing Services factors of the 12-Factor App methodology?
- **order-service changes:** I replaced the hard-coded connection value in `index.json` (previously `const TABBITMQ_URL = 'amqp://localhost';`) with an environment variable: `const RABBITMQ_CONNECTION_STRING = process.env.RABBITMQ_CONNECTION_STRING;`. This includes a fallback to localhost if the variable is not defined (which I don't expect to use in production). All uses of the original `RABBITMQ_URL` constant were updated to reference the environment variable. I also added a `.env` file and added the `dotenv` dependency to `package.json`.

- **product-service changes:** I replaced `main.rs` with the Lab 2 provided file, which uses `dotenv().ok()` and reads configuration values (including `PORT`) from environment variables: `let port: u16 = env::var("PORT")`.

Both changes comply with Factor 3 (Configurations) — storing config in the environment — and Factor 4 (Backing Services) — treating backing services as attached resources. This makes it easy to swap services without modifying application code; only environment variable values need updating. For example, to host `product-service` on another VM, update the connection URL used by the `store-front` to the appropriate public IP and port.

### Why is it important to use environment variables instead of hard-coding configurations in your application?
Hard-coded values make resource changes complex and error-prone. Using environment variables enables quick, smooth resource swaps without modifying application code; only environment variables need to be updated.

### Why is it important to have separate repositories for each microservice? How does this help maintain independence and scalability of each service?

Complying with Factor 1: Codebase
- A single codebase per app (tracked in revision control) isolates each service, improving independence and simplifying troubleshooting. This isolation prevents failures in one service from affecting others and allows each service to be developed and deployed independently.
- Separate repositories enable better scalability: each service can be containerized and scaled independently to meet demand, optimizing resources and cost.

Separate repositories enforce the **Code Base** principle, ensuring decoupled deployment lifecycles. This protects uptime and SLOs, allows tailored elasticity per service, and simplifies incident response with focused telemetry and runbooks, reducing MTTR.


# Additional Notes & Issues

**Setup:**
- `order-service-lab2-demo` repo
    - VM: `vm0-cst8915-lab2`
    - Public IP: 20.104.102.185
- RabbitMQ repo: `26W_CST8915-Lab1` (Ramy's repo)
    - VM: `vm2-2-cst8915-lab2`
    - Public IP: 20.220.219.133
- `product-service-lab2` repo
    - VM: `vm1-product-service`
    - Public IP: 20.151.107.149
- `store-front-lab2` repo
    - VM: `vm0-cst8915-lab2` (shared due to quota limitations)
    - Public IP: 20.220.219.133

**RabbitMQ VM Setup**
1. Create VM & add the inbound port range for rabbitMQ running on 15672 (Any protocol) (done)
2. SSH to VM using VSCode `ssh -i "C:\Users\Mimi\.ssh\mimi-key1.pem" mimi@20.220.218.231`
**Remote SSH issue:** I could not connect to other remote servers via VSCode Remote SSH. I spent over an hour troubleshooting and suspect the Azure CLI was missing or corrupted. I discovered this while attempting to connect, opening the output panel (Ctrl+~) and selecting the 'Remote SSH' logs. The configuration files and public IPs were correct; only `20.104.102.185` connected successfully.
![Remote SSH Error Logs & Config File](/CST8915-lab2-petstore-12-factor/CST8915-lab2-petstore-12-factor/img/ssh-issue.png)  

The only reliable fix was to copy the SSH command from the Azure portal and paste my `.pem` file path into the provided input, then paste that command into my SSH config. The portal command worked even when my manually-constructed command did not; it's possible an invisible whitespace was present in my original command.
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


**Store-front VM Setup (shared with RabbitMQ VM due to quota limitations)**
1. Create VM and open inbound port 8080.
2. SSH to the VM: `ssh -i "C:\Users\Mimi\.ssh\mimi-key1.pem" mimi@20.220.218.231`
3. Clone `store-front-lab2` and run it. I encountered `permission denied` when cloning; I suspect the VM lacked an SSH key for Git operations. I generated a new key with `ssh-keygen` and added it to my GitHub account.
4. The VM sizes were too small for the full demo. I deleted the `product-service` VM and recreated it with a larger size (B2als) to increase RAM.
- Install and run the storefront:
    - `sudo apt install npm`
    - `npm install @vue/cli-service@latest`
    - `npm install`
    - `npm run serve`
- Visit `http://20.220.219.133:8080`


**Product-service VM Setup**
1. Clone the repo and install dependencies:
    - `sudo apt update`
    - `sudo apt install build-essential`
    - `curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh`
    - `source "$HOME/.cargo/env"`
2. Run with `cargo run`.

Product service running in B2als VM:
![running](/CST8915-lab2-petstore-12-factor/img/running.png)

**Order-service VM Setup**
1. This VM was created in Lab 1 and is already configured. For consistency:
    - `sudo apt update`
    - `curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -`
    - `sudo apt update`
    - `sudo apt install -y nodejs`
    - `npm install`
    - Run with `node index.js`

Order service running in VM:
![running order-service](/CST8915-lab2-petstore-12-factor/img/order-service-running.png)