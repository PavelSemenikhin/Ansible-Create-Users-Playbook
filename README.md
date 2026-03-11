## Purpose
This playbook provisions a new user under the `servers` group, creating their home directory, installing the provided SSH key, and granting passwordless `sudo` through a drop-in file at `/etc/sudoers.d`.

## Prerequisites
- Control node has access to the private SSH key declared in `inventory/hosts.yml` (`ansible_ssh_private_key_file`).
- Your account can become `root` on the target host (`become: true` is enabled).
- The developer has shared their public SSH key (`ssh-ed25519 AAAA... developer@example.com`).

## Layout
- `inventory/hosts.yml` lists the servers and the connection details (for example, `server1` uses `~/.ssh/id_ed25519_uapp` as the SSH key).
- `inventory/host_vars/<host>.yml` defines a `users` array; each item must include `name` and `key`.
- `playbooks/create_users.yml` iterates over `users`, creates the account, installs the SSH key, and writes the sudoers entry validated with `visudo`.

## Adding a developer
1. Open `inventory/host_vars/<host>.yml` for the target server.
2. Append the developer entry to the `users` list:
   ```yaml
   users:
     - name: developer
       key: "ssh-ed25519 AAAAB3NzaC1yc2EAAAADAQABAAABAQC..."
   ```
3. Optionally override the hosts with `--limit` or pass a different list via `-e users=@users.yml`.

## Running the playbook
- Default execution:
  ```
  ansible-playbook -i inventory/hosts.yml playbooks/create_users.yml
  ```
- Target a single host:
  ```
  ansible-playbook -i inventory/hosts.yml playbooks/create_users.yml --limit server1
  ```
- Use a custom list of users:
  ```
  ansible-playbook -i inventory/hosts.yml playbooks/create_users.yml -e users=@custom_users.yml
  ```

## Testing and verification
1. Dry run with `--check` before making changes:
   ```
   ansible-playbook -i inventory/hosts.yml playbooks/create_users.yml --check --limit server1
   ```

## Example `inventory/host_vars/server1.yml`
```yaml
users:
  - name: user1
    key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHJRn9SnhdlfnLj/PRS5hW4HrZgf7OfsUweVvXv99Ld7 user1@example.com"
```
