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
```
# Alternatif: Hydra ile Parola Kırma Örneği
# Bu komut, Kali Linux üzerinde popüler bir brute force aracı olan Hydra kullanılarak da gerçekleştirilebilir.
# hydra -L users.txt -P passwords.txt rdp://192.168.10.100 rdp -V

```

<img width="1665" height="855" alt="saldırı1" src="https://github.com/user-attachments/assets/f955edb2-ca54-490f-8d74-4fd09078b150" />


## 2. Tespit Aşaması (Blue Team)
Splunk arayüzünde Windows Security Logları incelendi. Özellikle EventCode=4625 (An account failed to log on) olaylarına odaklanıldı.

Saldırı trafiğini izlemek için öncelikle RDP başarısız girişlerine karşılık gelen Windows Event ID 4625'e odaklanıldı. Normalde dakikada tek tük görülen bu logların, saldırı anında yüzlerce kat artışı, tehdit göstergesidir.

Kullanılan SPL (Splunk Processing Language) Sorgusu:

```
index=windows sourcetype="WinEventLog:Security" EventCode=4625
| stats count by Source_Network_Address, Target_User_Name
| sort - count
```

## 3. Analiz Sonuçları
Kısa süre içerisinde tek bir kaynak IP adresinden (Kali Makinesi) yüzlerce başarısız giriş denemesi tespit edildi.

Saldırının hangi kullanıcı adlarına yönelik yapıldığı raporlandı.

## 🌟 Gelecekteki Geliştirmeler (Next Steps)

1.  **Korelasyon Kuralı Geliştirme:** Splunk Enterprise Security (ES) veya basit bir Alarm kuralı yazarak, 5 saniye içinde aynı kaynaktan (Source_Network_Address) gelen 10'dan fazla 4625 olayını otomatik olarak uyarı (alert) şeklinde tetiklemek.
2.  **Otomatik Engelleme (Active Response):** Saldırgan IP adresini tespit ettikten sonra, bu adresi Windows Güvenlik Duvarı'nda (Firewall) otomatik olarak engelleme (fail2ban benzeri) mekanizması entegre etmek.

## 📸 Ekran Görüntüleri

<img width="1188" height="530" alt="ad users" src="https://github.com/user-attachments/assets/5ff2c0e3-7f85-4d62-a9ea-407663586973" />


<img width="786" height="817" alt="eventvwr" src="https://github.com/user-attachments/assets/1d93e6d4-8c05-4207-9fc0-f9eb3fa72570" />

## Splunk ile Anomali Tespiti: 
Grafik, saldırı anında (Mon Dec 8, 2025) tek bir kaynak IP adresinden gelen başarısız oturum açma denemelerinin sayısının normalin çok üzerine çıktığını göstermektedir. Bu ani artış (spike), saldırının otomatik olarak tespit edildiğinin görsel kanıtıdır.

<img width="1475" height="885" alt="image" src="https://github.com/user-attachments/assets/da97cd22-268e-44b0-96d2-442aa23a3b89" />


<img width="1075" height="842" alt="4625" src="https://github.com/user-attachments/assets/02eeb0a9-c595-4741-b7dd-e623b8baf93d" />


<img width="1064" height="841" alt="4625k" src="https://github.com/user-attachments/assets/d94341c8-e84e-431d-b43e-41b322aa6bf1" />


<img width="1455" height="694" alt="saldırı" src="https://github.com/user-attachments/assets/b45e4b37-2e05-44c8-8a81-f924737ee133" />

