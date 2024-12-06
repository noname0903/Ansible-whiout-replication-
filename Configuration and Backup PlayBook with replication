---
- name: Configure MariaDB replication
  hosts: replication
  become: yes
  vars:
    master_host: rep1
    slave_host: rep2
    replication_user: replica
    replication_password: juniper@123
    backup_directory: /var/backups/mysql/
    root_password: juniper@123

  tasks:
    - name: Install curl (if not already installed)
      apt:
        name: curl
        state: present
        update_cache: yes

    - name: Download and run the MariaDB repository setup script
      shell: "curl -LsS https://r.mariadb.com/downloads/mariadb_repo_setup | bash -s -- --mariadb-server-version='mariadb-11.6'"
      args:
        creates: /etc/apt/sources.list.d/mariadb.list

    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install MariaDB server
      apt:
        name: mariadb-server
        state: latest

    - name: Start and enable MariaDB service
      systemd:
        name: mariadb
        state: started
        enabled: yes

   - name: Set root password
      mysql_user:
        name: root
        password: "{{ root_password }}"
        host: "%"
        state: present

    - name: Remove test database
      mysql_db:
        name: test
        state: absent

    - name: Remove anonymous users
      mysql_user:
        name: ''
        host: "%"
        state: absent

    - name: Disable remote root login
      mysql_user:
        name: root
        host: "%"
        state: absent
    - name: Configure MariaDB for replication
      copy:
        dest: /etc/mysql/my.cnf
        content: |
          [mysqld]
          server-id = {{ hostvars[inventory_hostname]['server_id'] }}
          {% if inventory_hostname != master_host %}
          relay-log = /var/log/mysql/mysql-relay-bin.log
          read_only = 1
          {% else %}
          log_bin = /var/log/mysql/mysql-bin.log
          bind-address = 0.0.0.0
          {% endif %}
          log_error = /var/log/mysql/error.log
          max_connections = 100
          key_buffer_size = 16M
      owner: root
      group: root
      mode: '0644'

    - name: Restart MariaDB service
      service:
        name: mariadb
        state: restarted
        enabled: yes

    - name: Create replication user (only on master)
      mysql_user:
        name: "{{ replication_user }}"
        password: "{{ replication_password }}"
        host: "%"
        priv: "*.*:REPLICATION SLAVE"
        state: present
      when: inventory_hostname == master_host

    - name: Ensure backup directory exists (only on master)
      file:
        path: "{{ backup_directory }}"
        state: directory
        owner: root
        group: root
        mode: '0755'
      when: inventory_hostname == master_host

    - name: Create MySQL backup with date in filename (only on master)
      shell: |
        backup_dir="{{ backup_directory }}" filename="master_backup_$(date +\%Y-\%m-\%d).sql"
        /usr/bin/mysqldump -u root -p'{{ replication_password }}' --all-databases --master-data=2 --single-transaction --quick --lock-tables=false > "$backup_dir/$filename"
      when: inventory_hostname == master_host

    - name: Find the latest MySQL backup file (only on master)
      shell: "ls -t {{ backup_directory }}*.sql | head -n 1"
      register: latest_backup_file
      when: inventory_hostname == master_host

    - name: Debug the latest backup file
      debug:
        var: latest_backup_file
      when: inventory_hostname == master_host

    - name: Download the backup file from master (on slaves)
      fetch:
        src: "{{ latest_backup_file.stdout }}"
        dest: "{{ backup_directory }}"
        flat: yes
      when: inventory_hostname == master_host

    - name: Import the backup file to slave
      shell: |
        mysql -u root -p'{{ replication_password }}' < "{{ backup_directory }}/{{ latest_backup_file.stdout | basename }}"
      when: inventory_hostname == master_host

    - name: Read log_file and log_pos from backup (only on master)
      shell: |
        grep -m 1 "CHANGE MASTER TO" {{ latest_backup_file.stdout }} |
        grep -oE "MASTER_LOG_FILE='[^']*'|MASTER_LOG_POS=[0-9]+" |
        awk -F"=" '{print $2}' | tr -d "'"
      register: log_values
      when: inventory_hostname == master_host

    - name: Share log_file and log_pos to all hosts
      set_fact:
        log_file: "{{ log_values.stdout_lines[0] | default('') }}"
        log_pos: "{{ log_values.stdout_lines[1] | default('') }}"

    - name: Stop slave process
      mysql_replication:
        mode: stopreplica
        login_user: root
        login_password: "{{ replication_password }}"
      when: inventory_hostname == slave_host

    - name: Configure replication on slave
      command: >
        mariadb -u root -p'{{ replication_password }}' -e
        "CHANGE MASTER TO MASTER_HOST='{{ hostvars[master_host]['ansible_host'] }}',
        MASTER_USER='{{ replication_user }}',
        MASTER_PASSWORD='{{ replication_password }}',
        MASTER_LOG_FILE='{{ hostvars[master_host]['log_file'] }}',
        MASTER_LOG_POS={{ hostvars[master_host]['log_pos'] }};"
      when: inventory_hostname == slave_host

    - name: Start slave process
      mysql_replication:
        mode: startreplica
        login_user: root
        login_password: "{{ replication_password }}"
      when: inventory_hostname == slave_host

    - name: Check slave replication status
      mysql_replication:
        mode: getreplica
        login_user: root
        login_password: "{{ replication_password }}"
      register: slave_status
      when: inventory_hostname == slave_host

    - name: Display slave replication status
      debug:
        var: slave_status
      when: inventory_hostname == slave_host
