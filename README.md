# IdlePay Docker Compose

A community-maintained Docker Compose deployment for IdlePay, designed for Linux systems, self-hosted platforms and CrankkOS devices.

Register for an IdlePay account using my referral link if you appreciate my work:

### https://idlepay.io/?ref=5079E44D

---

## Why this repository exists

Some self-hosted platforms make deploying containers through Docker Compose or Portainer easier than using `docker run`. This repository provides ready-to-use configurations for those environments, including a dedicated configuration for CrankkOS, where persistent data must be stored under `/data`.

Both configurations use the official IdlePay Docker image.

---

## 1. General installation

### Supported environments

- Ubuntu and Debian
- Raspberry Pi OS
- CrankkIMG
- UmbrelOS using Portainer
- Other Linux systems supporting Docker Compose

**Tested on:** UmbrelOS, Raspberry Pi OS Lite on a Raspberry Pi 3, and CrankkIMG on an MNTD Gold miner.

### Requirements

- Docker and Docker Compose.
- An IdlePay account.
- An enrollment key generated in your IdlePay dashboard.

**UmbrelOS users:** Portainer must be installed and running.

### Step 1 — Download the files

Create an installation directory and download the configuration files directly from this repository:

```bash
mkdir -p ~/idlepay
cd ~/idlepay

wget -O docker-compose.yml \
  https://raw.githubusercontent.com/C-Man-The-Man/idlepay-d_compose/main/docker-compose.yml

wget -O .env \
  https://raw.githubusercontent.com/C-Man-The-Man/idlepay-d_compose/main/.env.example
```

Both files are now in `~/idlepay`, with the required filenames:

- `docker-compose.yml` — Docker Compose configuration.
- `.env` — environment variables and enrollment key.

If you prefer another installation directory, replace `~/idlepay` with your chosen path.

### Step 2 — Configure IdlePay

Edit the `.env` file:

```bash
nano .env
```

Its initial contents are:

```dotenv
IDLEPAY_HOSTNAME=CHOOSE_A_NAME
IDLEPAY_KEY=ipk_REPLACE_WITH_YOUR_ENROLLMENT_KEY
```

Replace the placeholders:

- `IDLEPAY_HOSTNAME`: choose a descriptive name that will appear in your Idlepay dashboard.
- `IDLEPAY_KEY`: enter your actual enrollment key.

To generate an enrollment key, open the **IdlePay dashboard**, select **Devices**, and use the **Enrollment keys** section.

`CTRL+O` and `ENTER` to save the file, `CTRL+X` to exit the editor.

### Step 3 — Start IdlePay

From the installation directory, run:

```bash
docker-compose up -d
```

Check the logs:

```bash
docker-compose logs -f
```

Successful startup should produce messages indicating that IdlePay is running and connecting to an IdlePay gateway.

Press `Ctrl+C` to stop following the logs. The container will continue running.

Useful commands:

```bash
# Check container status
docker ps -a --filter name=idlepay

# Stop and remove the container
docker-compose down

# Start it again
docker-compose up -d
```

The Docker-managed volume is explicitly named `idlepay-data`. Removing the container with `docker-compose down` does not remove this volume.

### Using Portainer on UmbrelOS

1. Install and start Portainer on UmbrelOS.
2. Open Portainer and select your Docker environment.
3. Navigate to **Stacks** and create a stack named `idlepay`.
4. Use the downloaded `docker-compose.yml` as the stack configuration. If using the Web editor, copy the contents of that file into the editor.
5. Add `IDLEPAY_HOSTNAME` and `IDLEPAY_KEY` through Portainer's environment-variable interface.
6. Deploy the stack.

If you download the files onto the Umbrel host, they will be available in the directory you selected. Portainer's Web editor does not automatically load a file simply because it exists in a host directory.

---

## 2. CrankkOS installation

This configuration is intended for CrankkOS devices, including Linux-based gateways and miners such as Linxdot, Bobcat, Nebra RockPi, Browan MerryIoT and PantherX2.

Hardware compatibility depends on the device's operating system, processor architecture and Docker environment. Not every device model has been independently tested.

### Important: CrankkOS storage

CrankkOS restricts persistent writable storage to `/data`.

The CrankkOS Compose configuration mounts the host directory `/data/idlepay` into the container at `/etc/idlepay`. This preserves IdlePay's expected internal configuration path while keeping persistent files on the host's writable partition.

### Step 1 — Create the installation directory

Connect to your CrankkOS device and run:

```bash
mkdir -p /data/idlepay && cd /data/idlepay
```

### Step 2 — Download the files

Download the CrankkOS-specific Compose configuration and the shared environment template:

```bash
wget -O docker-compose.yml \
  https://raw.githubusercontent.com/C-Man-The-Man/idlepay-d_compose/main/docker-compose-crankkos.yml

wget -O .env \
  https://raw.githubusercontent.com/C-Man-The-Man/idlepay-d_compose/main/.env.example
```

Although the repository stores the CrankkOS configuration as `docker-compose-crankkos.yml`, it is saved locally as `docker-compose.yml` so that the deployment commands remain the same across platforms.

Your `/data/idlepay` directory should now contain (run `ls -a`):

- `docker-compose.yml`
- `.env`

### Step 3 — Configure your device

Edit the environment file:

```bash
nano .env
```

Set your device name and enrollment key:

```dotenv
IDLEPAY_HOSTNAME=CHOOSE_A_NAME
IDLEPAY_KEY=ipk_REPLACE_WITH_YOUR_ENROLLMENT_KEY
```

Choose a hostname to be displayed in the Idlepay dashboard.

Generate the enrollment key from the **Devices → Enrollment keys** section of your **IdlePay dashboard**.

Save the file and exit the editor.

### Step 4 — Start IdlePay

From `/data/idlepay`, run:

```bash
docker-compose up -d
```

Check the logs:

```bash
docker-compose logs -f
```

Check the container status:

```bash
docker ps -a --filter name=idlepay
```

To stop and remove the container:

```bash
docker-compose down
```

The data under `/data/idlepay` remains on the host after the container is removed.

---

## Troubleshooting

### The device appears offline

Inspect the recent logs:

```bash
docker logs --tail 200 idlepay
```

Check that the enrollment key is correct and that the device can reach IdlePay's services.

### The device is marked "same IP" or "duplicate"

IdlePay identifies connections sharing a public IP address. According to its documented behavior, only one connection per public IP is used for earning.

Running additional containers behind the same public IP does not multiply earnings.

### The container exits unexpectedly

Inspect its exit status:

```bash
docker inspect idlepay --format \
  'Status={{.State.Status}} ExitCode={{.State.ExitCode}} OOMKilled={{.State.OOMKilled}} Error={{.State.Error}}'
```

Both configurations use Docker's `always` restart policy. Docker will attempt to restart the container after an exit, but the restart policy does not prevent the underlying application or host problem from recurring.

### Persistent data and device identity

Do not delete the persistent data directory or volume unless you understand the consequences.

IdlePay's configuration can contain device identity and authentication information. Do not clone an enrolled device's configuration to create another independent device.

---

## Official resources

- [IdlePay website](https://idlepay.io/)
- [Official Docker download instructions](https://idlepay.io/download)
- [IdlePay documentation](https://idlepay.io/docs)
- [IdlePay troubleshooting](https://idlepay.io/docs/troubleshooting)

---

## Disclaimer

This is a community-maintained Docker Compose project. It is not an official IdlePay product or an official Umbrel App Store integration. IdlePay's container image and service remain the property of their respective owners.

---

## Donations

**Bitcoin wallet address**
```text
bc1qpcfex53u7mqx4dc25gw7j7446amw9vn6743cn5
```

**EVM / Metamask  (ETH, ETC, OCTA, POL, PEAQ, MONAD, BASE etc.)**
```text
0xbE4879888d95B02B2FCaed2FcAeBbcf36829BDC9
```

**Solana wallet address**
```text
7EHWvShXfjLJ2HhzTf4CsHgjKckivfMQMjnEoUAEqau
```

**Sui wallet address**
```text
0x421a5a462f99c2d675d035d0c741ba5765a47c1e28f95d33ad770cd34a36a6ea
```

**Thank you for your support!**
