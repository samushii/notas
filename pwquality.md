- /etc/pam.d/passwd
	password required pam_pwquality.so retry=3

- /etc/security/pwquality.conf [configurações de senha]


{chage -M 90 <username>} [senhas expiram após certo tempo]

