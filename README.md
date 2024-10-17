# GitHub Action for Tencent Cloud CDN certificate deployment

Deploy Cloudflare SSL certificate to Tencent Cloud CDN.

## Usage

This action will deploy your PEM-formatted SSL certificate to Tencent Cloud CDN.

### Set Github Secrets
* CF_TOKEN
* CF_ZONE_ID
* EMAIL
* DOMAIN
* QCLOUD_SECRET_ID
* QCLOUD_SECRET_KEY

```yaml

name: auto certificate
on:
  schedule:
    - cron: '0 0 1 * *'
  workflow_dispatch:
jobs:
  deploy-to-qcloud-cdn:
    name: Deploy cloudflare certificate to Tencent Cloud CDN
    
    runs-on: ubuntu-latest
    steps:
      - name: get acme
        env:
          CF_TOKEN: ${{ secrets.CF_TOKEN }}
          CF_ZONE_ID: ${{ secrets.CF_ZONE_ID }}
          EMAIL: ${{ secrets.EMAIL }}
          DOMAIN: ${{ secrets.DOMAIN }}
        run: |
          curl https://get.acme.sh | sh -s email=${EMAIL}
          export CF_Token=$CF_TOKEN
          export CF_Zone_ID=$CF_ZONE_ID
          bash ~/.acme.sh/acme.sh --issue --dns dns_cf -d ${DOMAIN} -d *.${DOMAIN}
          cd ~/.acme.sh/${DOMAIN}_ecc/
          echo "FILE_FULLCHAIN=$(pwd)/fullchain.cer" >> $GITHUB_ENV
          echo "FILE_KEY=$(pwd)/${DOMAIN}.key" >> $GITHUB_ENV

      - uses: lietxia/cf-certificate-to-tencentcloud@v2
        with:
          # Use Access Key
          secret-id: ${{ secrets.QCLOUD_SECRET_ID }}
          secret-key: ${{ secrets.QCLOUD_SECRET_KEY }}

          # Specify PEM fullchain file
          fullchain-file: ${{ env.FILE_FULLCHAIN }}
          # Specify PEM private key file
          key-file: ${{ env.FILE_KEY }}

          # Deploy to CDN
          cdn-domains: |
            www.r-mj.com
            rate.r-mj.com
            cdn.r-mj.com
```

## Permissions

Please make sure the Tencent Cloud account you use has the following permissions:

- QcloudCDNFullAccess
- QcloudSSLFullAccess
