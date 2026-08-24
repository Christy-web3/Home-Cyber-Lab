# Publish this project on GitHub

## Before publishing

1. Read every file and confirm it contains no passwords, tokens, public IP addresses, university-only hostnames, or personal identifiers you do not want public.
2. Add only sanitised screenshots. Crop or redact browser sessions, cookies, credentials, email addresses, and unrelated desktop content.
3. Keep VM disk images, packet captures, private keys, and raw logs out of Git.

## Option A: GitHub website and Git commands

1. Sign in to GitHub and select **New repository**.
2. Name it `home-cyber-lab`.
3. Add a short description, for example: `Documented pfSense, Wazuh, Ubuntu and Kali defensive-security lab.`
4. Select **Public**.
5. Do not add a README, `.gitignore`, or licence on GitHub because this project already includes them.
6. Create the repository.

From inside the local `home-cyber-lab` folder, run:

```bash
git init -b main
git add .
git status
git commit -m "Initial Home Cyber Lab documentation"
git remote add origin https://github.com/YOUR-USERNAME/home-cyber-lab.git
git push -u origin main
```

Replace `YOUR-USERNAME` with your GitHub username.

## Option B: GitHub CLI

If GitHub CLI is installed and authenticated:

```bash
git init -b main
git add .
git commit -m "Initial Home Cyber Lab documentation"
gh repo create home-cyber-lab --public --source=. --remote=origin --push
```

## Recommended repository settings

- Enable secret scanning if available for the account.
- Enable dependency alerts if code or packages are added later.
- Add repository topics such as `cybersecurity`, `wazuh`, `pfsense`, `virtualbox`, `siem`, `linux`, and `home-lab`.
- Review every future screenshot before committing it.

## Suggested first release

After reviewing the published repository, create a release named `v0.1-lab-foundation` with notes covering:

- Initial network architecture
- Wazuh SSH detection
- Fail2Ban investigation
- Command reference and lab notes

