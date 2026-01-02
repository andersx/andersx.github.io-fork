---
layout: post
title: "Opsætning af Unifi Gateway (og IPv6) hos NetNørden"
lang: "da"
date: 2025-04-10
categories: guides networking unifi ubiquiti router ipv6 netnoerden netnørden
---
Dette er en kort guide til opsætning af en Unifi Gateway (inkl. IPv6) med fiberforbindelse via den ny udbyder NetNørden.

Jeg bruger i denne guide en **Unifi UCG-Ultra** med **UniFi OS version 4.1.13** og **Unifi Network version 9.0.114**.
Routeren er tilsluttet direkte fra sit **WAN-stik til en LAN-port** i fiberboksen fra TDC.

Alt konfigureres direkte i UniFi Network-appen, og det var **ikke nødvendigt at genstarte routeren** undervejs — heller ikke ved skiftet fra min tidligere udbyder (Hiper) til NetNørden.

Du kan også trykke her for at se min guide til [**opsætning af IPv6 hos Hiper**](https://andersx.dk/guides/networking/ubiquiti/2025/03/16/dk-unifi-hiper.html).

---

## Opsætning af WAN med IPv4 og IPv6

1. Log ind på din UniFi Controller og gå til **Settings** > **Internet**
2. Vælg **Primary (WAN1)** og klik på **Manual**

3. **VLAN ID** skal være slået fra. (De fleste udbydere kræver VLAN ID = 101, men NetNørden gør ikke.)

4. Din **IPv4 Configuration** burde virke med indstillingen fra **Auto**, intet at ændre her.
5. Under **IPv6 Configuration**, vælg:
   - **IPv6 Connection**: `DHCPv6`
   - **Prefix Delegation Size**: ~~`48` (NetNørden bruger /48)~~
     - `Auto` - NetNørden tildeler dynamisk prefix `/56` (Updated December 2025)
   - **DNS Servers**: `Auto`

6. Klik på **Apply Changes** og vent — din router bør få tildelt en offentlig IPv6-adresse med det samme.
   Jeg skulle hverken genstarte eller unplugge routeren, før det virkede - fantastisk!

![Unifi Controller - WAN setup]({{ site.baseurl }}/assets/images/netnoerden-ipv6-wan.png)

---

## Opsætning af LAN med IPv4 og IPv6

1. Gå til **Settings** > **Networks**, og vælg dit LAN-netværk i listen (typisk kaldet **Default**)
2. IPv4-opsætningen burde virke med **Auto**
3. Gå til **IPv6**, vælg **Prefix Delegation**, og lad resten af indstillingerne være uændrede
  - **Advanced** kan bare stilles til **Auto**, så burde det køre fint med [SLAAC](https://en.wikipedia.org/wiki/IPv6_address#Stateless_address_autoconfiguration_(SLAAC)) på dit LAN.

![Unifi Controller - LAN setup]({{ site.baseurl }}/assets/images/netnoerden-ipv6-lan.png)

Det burde være alt der skal til!

---

## Test opsætningen

- Check at din router har fået tildelt en offentlig WAN IPv6 adresse på forsiden af Unifi Network. Hvis du har fast IP burde du kunne se den IPv4 og IPv6 du har fået tildelt i din velkomst email.

![Unifi Controller]({{ site.baseurl }}/assets/images/unifi-netnoerden-landing-page.png)

---

- Check at din computer har fået en IPv6 adresse f.eks. `nmcli device show | grep IP6` i terminalen:

```
$ nmcli device show | grep IP6
IP6.ADDRESS[1]:                         ****:****:****:****:***:****:****:****/64
IP6.ADDRESS[2]:                         ****:****:****:****:****:****:****:****/64
IP6.ADDRESS[3]:                         ****::****:****:****:****/64
IP6.GATEWAY:                            ****::****:****:****:****
IP6.ROUTE[1]:                           dst = ****::/64, nh = ::, mt = 1024
IP6.ROUTE[2]:                           dst = ****:****:****:****::/64, nh = ::, mt = 100
IP6.ROUTE[3]:                           dst = ::/0, nh = ****::****:****:****:****, mt = 100
IP6.DNS[1]:                             ****:****:****:****::1
IP6.GATEWAY:                            --
IP6.GATEWAY:                            --
IP6.ADDRESS[1]:                         ::1/128
IP6.GATEWAY:                            --
IP6.ROUTE[1]:                           dst = ::1/128, nh = ::, mt = 256

```

---

For at teste at IPv6 virker korrekt kan du f.eks. bruge [sikkerpånettet.dk/connection/](https://da.sikkerpånettet.dk/connection/).

![Sikker På Nettet]({{ site.baseurl }}/assets/images/sikker-paa-nettet-ipv6.png)

*Glædelig IPv6!*
