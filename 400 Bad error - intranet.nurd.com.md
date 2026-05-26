Redis integration for intranet.nurd.com
1. Verify Redis is already running

No need to install again if Redis is already configured:

systemctl status redis-server

Verify:

redis-cli ping

Expected:

PONG
2. Edit oauth2-proxy service for intranet

Open the service:

vi /etc/systemd/system/oauth2-proxy-intranet.nurd.com.service

Update ExecStart:

[Unit]
Description=oauth2-proxy-intranet.nurd.com

[Service]
ExecStart=/usr/bin/oauth2-proxy \
  --alpha-config /etc/oauth2/intranet.nurd.com.yaml \
  --session-store-type=redis \
  --redis-connection-url=redis://127.0.0.1:6379

Environment=OAUTH2_PROXY_COOKIE_SECRET=xxxxxxxxxxxx
Environment=OAUTH2_PROXY_PROMPT=select_account
Environment=OAUTH2_PROXY_HTTP_ADDRESS=0.0.0.0:2214
Environment=OAUTH2_PROXY_EMAIL_DOMAINS=*
Restart=always
LimitNOFILE=10048576

[Install]
WantedBy=multi-user.target

Changes from Jira site:

Config file:

/etc/oauth2/intranet.nurd.com.yaml

Service:

oauth2-proxy-intranet.nurd.com

Port:

2214

(keep existing port if different)

3. Reload and restart
systemctl daemon-reload
systemctl restart oauth2-proxy-intranet.nurd.com
4. Verify Redis activity

Monitor Redis:

redis-cli monitor

Then login to intranet.nurd.com.

Expected:

GET session:xxxx
SET session:xxxx
EXPIRE session:xxxx
5. Clear browser cookies

Delete cookies for:

intranet.nurd.com

Login again.

Expected result

Before:

Browser → Large OAuth cookie → Nginx 400 → page/UI issue

After:

Browser → Small session ID cookie → Redis stores session → Nginx accepts request → site works

This reuses the same Redis instance and adds almost no additional resource usage. No separate Redis installation is needed.
