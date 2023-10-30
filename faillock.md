- /etc/pam.d/system-auth

	2 auth 	required	pam_faillock.so preauth silent audit deny=3 unlock_time=600 [even_deny_root]
	4 auth	[default=die]	pam_faillock.so authfail audit deny=3 unlock_time=600

- /etc/pam.d/password-auth
