1\. Two Types of Connection (ansible Master \& Slave)



Keyless-1



ssh-copy-id -o IdentityFile=<path-to-pem> ubuntu@<slave-ip>

ssh ubuntu@<slave-ip>

ansible all -m ping

========================================================================

ssh key password-2



Master

\# ssh -i ~/key.pem ubuntu@<slave-ip>

ssh -i <path-to-key.pem> ubuntu@<slave-ip>



Slave



sudo vim /etc/ssh/sshd\_config.d/60-cloudimg-settings.conf

\# → PasswordAuthentication yes



sudo vim /etc/ssh/sshd\_config

\# → PasswordAuthentication yes



sudo systemctl restart ssh

sudo passwd ubuntu





Master

ssh-copy-id ubuntu@<slave-ip>

ssh ubuntu@<slave-ip>  # should NOT ask password

ansible all -m ping    # pong expected



====================================================================================================================================================================================================



2\. Adhoc commands | Inventory (File path)-> /etc/ansible/hosts



cd /etc/ansible/hosts



vim inventory.ini

------------------------------------------

\[slave]

3.23.130.165 ansible\_user=ubuntu

------------------------------------------



ansible all -i inventory.ini -m shell -a "sudo apt install openjdk-11-jdk -y"



ansible -i inventory.ini -m ping all



====================================================================================================================================================================================================



3\. Ansible PlayBook



Create Playbook — site.yml

-------------------------------------------

---

\- hosts: all

&nbsp; become: true

&nbsp; tasks:

&nbsp;   - name: Install Apache2

&nbsp;     ansible.builtin.apt:

&nbsp;       name: apache2         # 👈 Typo fixed: "apche2" → "apache2"

&nbsp;       state: present

&nbsp;       update\_cache: yes



&nbsp;   - name: Copy index.html to web root

&nbsp;     ansible.builtin.copy:

&nbsp;       src: index.html

&nbsp;       dest: /var/www/html/index.html

&nbsp;       owner: root

&nbsp;       group: root

&nbsp;       mode: '0644'





-------------------------------------------



ansible-playbook -i inventory.ini site.yml





====================================================================================================================================================================================================



4\. Ansible Role



ansible-galaxy role init apache\_role



This will generate a fully structured folder like this:

apache\_role/

├── defaults/

│   └── main.yml

├── files/

├── handlers/

│   └── main.yml

├── meta/

│   └── main.yml

├── tasks/

│   └── main.yml       <-- 💖 This is where you write your playbook tasks

├── templates/

├── tests/

│   ├── inventory

│   └── test.yml

└── vars/

&nbsp;   └── main.yml

----------------------------------------------------------------------------



ls -lrt



**(cut task from Playbook — site.yml and paste here)**



vim apache\_role/tasks/main.yml

-----------------------------------------------------------------------------

&nbsp;   - name: Install Apache2

&nbsp;     ansible.builtin.apt:

&nbsp;       name: apache2         # 👈 Typo fixed: "apche2" → "apache2"

&nbsp;       state: present

&nbsp;       update\_cache: yes



&nbsp;   - name: Copy index.html to web root

&nbsp;     ansible.builtin.copy:

&nbsp;       src: files/index.html

&nbsp;       dest: /var/www/html/index.html

&nbsp;       owner: root

&nbsp;       group: root

&nbsp;       mode: '0644'

-----------------------------------------------------------------------------



Playbook — site.yml

-----------------------------------------------------------------------------

---

\- hosts: all

&nbsp; become: true

&nbsp; roles:

&nbsp;   - apache\_role



-----------------------------------------------------------------------------



apache\_role/files/index.html

-----------------------------------------------------------------------------

\- name: Copy index.html to web root

&nbsp; ansible.builtin.copy:

&nbsp;   src: index.html                  # Comes from 'files/' automatically

&nbsp;   dest: /var/www/html/index.html

&nbsp;   owner: root

&nbsp;   group: root

&nbsp;   mode: '0644'

-----------------------------------------------------------------------------



ansible-playbook -i inventory.ini site.yml



====================================================================================================================================================================================================



5\.  Reusing Ansible Roles from Galaxy



ansible-galaxy install bsmeding.docker

~/.ansible/roles/bsmeding.docker



vim docker\_playbook.ym

-------------------------------------------------------------------------------

---

\- hosts: all

&nbsp; become: true

&nbsp; roles:

&nbsp;   - bsmeding.docker

-------------------------------------------------------------------------------



ansible-playbook -i inventory.ini docker\_playbook.yml



