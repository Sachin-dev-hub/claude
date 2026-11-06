# Ansible — Interview Q&A (incl. Windows)

**JD KEY SKILL: "Ansible with Windows servers pipeline setup and deployment" — be ready for this.**

---

## Q1. What is Ansible? How is it different from Chef/Puppet?

- Ansible = **configuration management + automation** tool.
- **Agentless** — no software installed on target machines. Connects via **SSH** (Linux) or **WinRM** (Windows).
- Uses **YAML** playbooks (easy to read).
- **Idempotent** — run the same playbook 100 times, result is the same.
- Chef/Puppet need agents on each machine (agent-based). Ansible is simpler and agentless.

---

## Q2. Key concepts — inventory, playbook, module, handler, role?

- **Inventory:** list of servers you manage. Static (INI/YAML file) or **dynamic** (from AWS EC2 using tags — auto-discovers servers).
- **Playbook:** the YAML file with the tasks to run.
- **Module:** a ready-made task, e.g., `yum`, `copy`, `service`, `win_service`. You call modules in tasks.
- **Handler:** runs only when a task makes a change — e.g., restart nginx only if config file changed.
- **Role:** a reusable package (folder structure: tasks, vars, handlers, templates, defaults). A playbook can use many roles. Like Terraform modules — write once, reuse everywhere.

---

## Q3. How does Ansible work with Windows servers? (KEY — practice this well)

- Ansible is agentless, so instead of SSH it connects to Windows using **WinRM** (Windows Remote Management).

**Control machine (Linux) side:**
- Install `pywinrm` Python package.

**Windows side setup (one time per server):**
1. Enable WinRM: run `Enable-PSRemoting` in PowerShell.
2. Create a self-signed certificate and set up HTTPS WinRM on port **5986**.
3. Open the firewall for port 5986.
4. Have a Windows user (local or domain) with admin rights for Ansible.

**Inventory config for Windows:**
```ini
[windows]
win-server-01 ansible_host=10.0.1.5

[windows:vars]
ansible_connection=winrm
ansible_winrm_transport=ntlm
ansible_winrm_server_cert_validation=ignore
ansible_user=Administrator
ansible_password=SecretPass
```

**Test:** `ansible windows -m win_ping`

**Windows modules to know:**
- `win_service` (start/stop/restart services)
- `win_copy` (copy files to Windows)
- `win_file` (create/delete files and folders)
- `win_shell` (run PowerShell commands)
- `win_updates` (Windows patching)
- `win_chocolatey` (install software)
- `win_iis_webapppool` (IIS management)
- `win_feature` (enable Windows features/roles)

**Deployment pipeline example (matches JD):**
Ansible playbook pushes an MSI/EXE/zip to the Windows server (`win_copy`), runs the installer (`win_shell`), restarts the service (`win_service`), and validates it's running — all from Jenkins after the build.

---

## Q4. What does "idempotent" mean? Give an example.

- Running a playbook many times gives the same end state — no side effects on repeat runs.
- **Example:** a `win_service` task with `state: started` — if the service is already running, Ansible does nothing; if stopped, it starts it. Result is always "service is running."
- This is why Ansible is safe to run repeatedly and in scheduled automation.

---

## Q5. How do you pass variables? What is Ansible Vault?

**Ways to define:** inventory, playbook `vars:`, role defaults, `--extra-vars` on command line, vault.
**Precedence:** command-line `--extra-vars` wins over everything, then playbook vars, then inventory.

**Ansible Vault:** encrypts sensitive values (passwords, keys) in playbooks:
- `ansible-vault encrypt secrets.yml`
- Run with `--ask-vault-pass`
- (JD mentions Vault/Secrets Manager — mention this.)

---

## Q6. What are ad-hoc commands?

- Run a single task without a playbook:
  - `ansible all -m ping`
  - `ansible windows -m win_shell -a "Get-Service spooler"`
- Good for quick checks, not for complex work (use playbooks for that).

---

## Q7. How do you run a playbook?

- `ansible-playbook site.yml -i inventory`
- Common flags: `--limit` (run on some hosts), `--tags`, `--extra-vars`, `--check` (dry run), `--ask-vault-pass`.

---

## Quick commands to remember

| Command | What it does |
|---|---|
| `ansible all -m ping` | Test Linux connection (SSH) |
| `ansible windows -m win_ping` | Test Windows connection (WinRM) |
| `ansible-playbook site.yml -i inventory` | Run a playbook |
| `ansible-playbook site.yml --check` | Dry run (no changes) |
| `ansible-vault encrypt secrets.yml` | Encrypt a secrets file |
| `ansible all -m setup` | Gather server facts |