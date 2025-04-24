![FA-IT Consulting Logo](https://fake-url.com/logo-fa-it.jpeg)

# 🛡️ Implémentation des protocoles de sécurisation de la messagerie : SPF, DKIM, DMARC, BIMI et SMTP

## 📌 Objectif
Protéger la messagerie contre :
- L’usurpation d’identité (spoofing)
- Le phishing
- L’altération de messages
- L’utilisation frauduleuse de marque

## 1. Fonction de chaque protocole

| Protocole | Rôle | Vérifie | Niveau | Aligné avec le From visible |
|----------|------|---------|--------|------------------------------|
| SMTP     | Protocole d’envoi de mail | Rien par défaut | Transport (enveloppe) | ❌ |
| SPF      | Autorise certaines IP à envoyer des mails pour un domaine | IP du serveur d’envoi via MAIL FROM | Enveloppe SMTP | ❌ |
| DKIM     | Signe le message avec une clé privée | En-têtes + corps du message | Header | ✅ |
| DMARC    | Vérifie SPF/DKIM + alignement avec le From | Alignement + politique | Policy | ✅ |
| BIMI     | Affiche un logo de marque | Repose sur DMARC | Branding | ✅ |

## 2. Vue technique (enveloppe vs en-tête)

(Diagramme à insérer)

## 3. Mise en œuvre avec Microsoft 365

### SPF
```
v=spf1 include:spf.protection.outlook.com -all
```

### DKIM
Activation dans Microsoft 365 Defender, création de deux enregistrements CNAME :
- selector1._domainkey.votredomaine.com
- selector2._domainkey.votredomaine.com

### DMARC
```
v=DMARC1; p=quarantine; rua=mailto:rapport@votredomaine.com; adkim=s; aspf=s
```

### BIMI
```
v=BIMI1; l=https://votredomaine.com/logo.svg; a=https://votredomaine.com/vmc.pem
```

## 4. Script PowerShell de vérification DNS

```powershell
$domain = "votredomaine.com"
Write-Host "🔍 Vérification DNS pour le domaine : $domain" -ForegroundColor Cyan
# SPF
$spf = (Resolve-DnsName -Name $domain -Type TXT | Where-Object {$_.Strings -like 'v=spf1*'}) | Select-Object -ExpandProperty Strings
Write-Host "`nSPF :" $spf -ForegroundColor Yellow
# DMARC
$dmarc = (Resolve-DnsName -Name "_dmarc.$domain" -Type TXT -ErrorAction SilentlyContinue).Strings
Write-Host "`nDMARC :" $dmarc -ForegroundColor Yellow
# DKIM
$selector1 = (Resolve-DnsName -Name "selector1._domainkey.$domain" -Type CNAME -ErrorAction SilentlyContinue)
$selector2 = (Resolve-DnsName -Name "selector2._domainkey.$domain" -Type CNAME -ErrorAction SilentlyContinue)
Write-Host "`nDKIM selector1 :" $selector1.NameHost -ForegroundColor Yellow
Write-Host "DKIM selector2 :" $selector2.NameHost -ForegroundColor Yellow
```

## 5. Modèle de documentation client

### SPF
```
v=spf1 include:spf.protection.outlook.com -all
```

### DKIM
Activation via Microsoft 365 et enregistrement CNAME des selectors

### DMARC
```
v=DMARC1; p=quarantine; rua=mailto:dmarc-report@votredomaine.com; adkim=s; aspf=s
```

### BIMI
```
v=BIMI1; l=https://votredomaine.com/logo.svg; a=https://votredomaine.com/vmc.pem
```

## 6. Outils de test recommandés

- SPF : https://mxtoolbox.com/spf.aspx
- DKIM : https://dkimcore.org/tools/
- DMARC : https://dmarcian.com/dmarc-inspector/
- BIMI : https://bimigroup.org/tools/

