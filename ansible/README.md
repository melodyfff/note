# ansible 快速入门

文件结构
```json
.
├── hosts
├── roles
│   └── test
│       ├── files
│       │   ├── check_file.sh
│       │   └── test.sh
│       ├── handlers
│       │   └── main.yml
│       ├── tasks
│       │   └── main.yml
│       ├── templates
│       │   ├── proxy.j2
│       │   └── test.j2
│       └── vars
│           └── main.yml
└── site.yml
```

just run 
```bash
ansible-playbook -i hosts -v site.yml
```