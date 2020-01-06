# Introduction
This package is an opinionated implementation of role based access control for NetBox.

It completely replaces the default authentication backend, using Active Directory for authentication and determining group membership. A user's roles are updated only on login, and are stored in the database.

# Installation
Clone this repository.

```
$ git clone https://github.com/ebusto/netbox-rbac /opt/netbox-rbac
```

Install the dependencies.

```
$ pip3 install --upgrade -r /opt/netbox-rbac/requirements.txt
```

Enable the Django "application".

```
$ ln -sf /opt/netbox-rbac/netbox_rbac /opt/netbox/netbox/netbox_rbac
```

# Configuration
Add the following to `settings.py`.

```
AUTHENTICATION_BACKENDS = [
    'netbox_rbac.backend.Backend',
]

REST_FRAMEWORK.update({
	'DEFAULT_PERMISSION_CLASSES': (
		'netbox_rbac.api.TokenPermissions',
		'netbox.api.TokenPermissions',
	)
})

INSTALLED_APPS += (
	'netbox_rbac',
)

MIDDLEWARE += (
	'netbox_rbac.middleware.Middleware',
)

TEMPLATES[0]['DIRS'].insert(0, os.path.join(BASE_DIR, 'netbox_rbac', 'templates'))

LOGGING.update({
	'loggers': {
		'netbox_rbac': {
			'handlers': ['console'],
			'level':     'INFO',
		},
	},
})

RBAC = {
	'AUTH': {
		'LDAP': {
			'domain': 'COMPANY.COM',
			'server': 'ldap://ldap.company.com:3268',
			'search': {
				'group': {
					'base':   'OU=Groups,DC=company,DC=com',
					'filter': '(&(sAMAccountName=%s)(objectClass=group))',
				},
				'member': {
					'base':   'OU=Accounts,DC=company,DC=com',
					'filter': '(&(sAMAccountName=%s)(memberOf:1.2.840.113556.1.4.1941:=%s))',
				},
				'user': {
					'base':   'OU=Accounts,DC=company,DC=com',
					'filter': '(&(sAMAccountName=%s)(objectClass=user))',
				},
			},
		},
	},
	'RULE': [
		'/opt/netbox-rules/rules.yaml',
		'https://rules.company.com/rules.yaml',
	],
}
```

Add the following to `urls.py`.
```
_patterns += [
	path('', include('netbox_rbac.urls') ),
]
```

# Database
Generate and apply RBAC model migrations.

```
$ ./manage.py makemigrations netbox_rbac
$ ./manage.py showmigrations
$ ./manage.py migrate
```

# Rules
See the [example](rules.yaml) rules, and [documentation](RULES.md).
