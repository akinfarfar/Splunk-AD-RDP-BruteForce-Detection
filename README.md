# Active Directory RDP Brute Force Attack & Splunk Detection

Bu proje, sanal bir Active Directory ortamında RDP Brute Force saldırısının simüle edilmesi ve Splunk SIEM aracı kullanılarak tespit edilmesini kapsamaktadır.

## 🎯 Proje Amacı
Gerçek dünya senaryolarına uygun olarak; bir saldırganın Active Directory ortamına sızma girişimini analiz etmek ve bu girişimi log yönetimi (SIEM) ile nasıl görünür hale getirebileceğimizi deneyimlemek.

## 🛠️ Kullanılan Teknolojiler ve Mimari

| Bileşen | Teknoloji / Araç | Açıklama |
| :--- | :--- | :--- |
| **SIEM** | Splunk Enterprise | Log toplama, indeksleme ve görselleştirme. |
| **Saldırgan** | Kali Linux (xfreerdp) | Brute force saldırısını gerçekleştiren makine. |
| **Hedef** | Windows 10 / Server (AD Üyesi) | RDP servisi açık, saldırıya uğrayan makine. |
| **Log Agent** | Splunk Universal Forwarder | Windows loglarını Splunk'a iletir. |

## 🚀 Uygulama Adımları

### 1. Saldırı Aşaması (Red Team)
Kali Linux üzerinden `xfreerdp` aracı kullanılarak hedef IP adresine (Örn: 192.168.10.100) saldırı başlatıldı.
```bash
# Kullanılan Örnek Komut
for p in $(cat passwords.txt); do
    echo "Trying $p"
    xfreerdp /v:192.168.10.100 /u:jsmith /p:$p /cert:ignore /timeout:2000
done
```

<img width="1665" height="855" alt="saldırı1" src="https://github.com/user-attachments/assets/f955edb2-ca54-490f-8d74-4fd09078b150" />


## 2. Tespit Aşaması (Blue Team)
Splunk arayüzünde Windows Security Logları incelendi. Özellikle EventCode=4625 (An account failed to log on) olaylarına odaklanıldı.

Kullanılan SPL (Splunk Processing Language) Sorgusu:

```
index=windows sourcetype="WinEventLog:Security" EventCode=4625
| stats count by Source_Network_Address, Target_User_Name
| sort - count
```

## 3. Analiz Sonuçları
Kısa süre içerisinde tek bir kaynak IP adresinden (Kali Makinesi) yüzlerce başarısız giriş denemesi tespit edildi.

Saldırının hangi kullanıcı adlarına yönelik yapıldığı raporlandı.

## 📸 Ekran Görüntüleri

<img width="1188" height="530" alt="ad users" src="https://github.com/user-attachments/assets/5ff2c0e3-7f85-4d62-a9ea-407663586973" />


<img width="786" height="817" alt="eventvwr" src="https://github.com/user-attachments/assets/1d93e6d4-8c05-4207-9fc0-f9eb3fa72570" />


<img width="674" height="738" alt="splunk" src="https://github.com/user-attachments/assets/5280c2cc-99b1-42f9-9db1-3a662f5824c0" />


<img width="1075" height="842" alt="4625" src="https://github.com/user-attachments/assets/02eeb0a9-c595-4741-b7dd-e623b8baf93d" />


<img width="1064" height="841" alt="4625k" src="https://github.com/user-attachments/assets/d94341c8-e84e-431d-b43e-41b322aa6bf1" />


<img width="1455" height="694" alt="saldırı" src="https://github.com/user-attachments/assets/b45e4b37-2e05-44c8-8a81-f924737ee133" />

