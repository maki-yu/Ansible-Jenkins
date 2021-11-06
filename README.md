# Ansible

- ディレクトリ構造
```
.
├── README.md
└── ansible
    ├── ansible.cfg
    ├── hosts
    ├── playbook.yml
    └── roles
        ├── common
        │   └── tasks
        │       └── main.yml
        ├── mysql
        │   └── tasks
        │       └── main.yml
        ├── nginx
        │   └── tasks
        │       └── main.yml
        ├── nodejs
        │   └── tasks
        │       └── main.yml
        └── ruby
            ├── tasks
            │   └── main.yml
            └── templates
                └── rbenv_system.sh.j2
```