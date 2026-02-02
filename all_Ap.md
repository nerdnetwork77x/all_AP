            ---------- AP no vlan----------

no dot11 mbssid                    # ปิดโหมด MBSSID เพื่อเลี่ยง Error

! --- 2. สร้างชื่อ Wi-Fi (SSID) ---
dot11 ssid MyHomeWiFi              # <--- ใส่ชื่อ Wi-Fi ที่ต้องการ
   authentication open 
   authentication key-management wpa version 2
   wpa-psk ascii 0 MyPassword123   # <--- ใส่รหัสผ่าน Wi-Fi (8 หลักขึ้นไป)
   guest-mode                      # สั่งให้ Broadcast ชื่อออกไป

! --- 3. ตั้งค่าคลื่น 2.4GHz (Radio 0) ---
interface Dot11Radio0
   no shutdown
   encryption mode ciphers aes-ccm # ตัวเข้ารหัส WPA2
   ssid MyHomeWiFi
   bridge-group 1                  # เชื่อมเสาเข้ากับสะพานภายในเครื่อง

! --- 4. ตั้งค่าคลื่น 5GHz (Radio 1) ---
interface Dot11Radio1
   no shutdown
   encryption mode ciphers aes-ccm # ตัวเข้ารหัส WPA2
   ssid MyHomeWiFi
   bridge-group 1                  # เชื่อมเสาเข้ากับสะพานภายในเครื่อง

! --- 5. ตั้งค่าพอร์ตสาย LAN (GigabitEthernet) ---
interface GigabitEthernet0
   no shutdown
   bridge-group 1                  # เชื่อมรู LAN เข้ากับสะพานภายในเครื่อง

! --- 6. ตั้งค่า IP ให้ตัว AP (เพื่อขอเน็ตจาก Router) ---
interface BVI1
   ip address dhcp                 # ให้ Router หลักแจก IP มาให้ (ง่ายที่สุด)
   no shutdown
----------------------------------------------------------------------------
            ------------ AP 2 WIFI ------------
conf t
  dot11 mbssid
  
dot11 ssid WIFI-STAFF
     vlan 10
     authentication open
     authentication key-management wpa version 2
     wpa-psk ascii 0 Staff@123
     mbssid guest-mode
  exit
  
dot11 ssid WIFI-GUEST
     vlan 20
     authentication open
     authentication key-management wpa version 2
     wpa-psk ascii 0 Guest@123
     mbssid guest-mode
  exit
  
interface Dot11Radio0
   no shutdown
   encryption vlan 10 mode ciphers aes-ccm
   encryption vlan 20 mode ciphers aes-ccm
   ssid WIFI-STAFF
   ssid WIFI-GUEST
  exit
  
interface Dot11Radio1
   no shutdown
   encryption vlan 10 mode ciphers aes-ccm
   encryption vlan 20 mode ciphers aes-ccm
   ssid WIFI-STAFF
   ssid WIFI-GUEST
  
interface GigabitEthernet0
   no shutdown
interface GigabitEthernet0.10
   encapsulation dot1Q 10
   bridge-group 10
interface GigabitEthernet0.20
   encapsulation dot1Q 20
   bridge-group 20
  
interface Dot11Radio0.10
   encapsulation dot1Q 10
   bridge-group 10
interface Dot11Radio0.20
   encapsulation dot1Q 20
   bridge-group 20
interface Dot11Radio1.10
   encapsulation dot1Q 10
   bridge-group 10
interface Dot11Radio1.20
   encapsulation dot1Q 20
--------------------------------------------
AP 2 wifi + nat + vlan +dhcp
Router

! --- กำหนดทางออกอินเทอร์เน็ต ---
interface GigabitEthernet0/1 (ขาที่ต่อกับ Modem)
 ip address dhcp
 ip nat outside
 no shutdown

! --- สร้าง Gateway ให้แต่ละ VLAN (Sub-interfaces) ---
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip nat inside

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip nat inside

! --- คำสั่ง NAT เพื่อให้ทุกวงออกเน็ตได้ ---
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
ip nat inside source list 1 interface GigabitEthernet0/1 overload

ip dhcp excluded-address 192.168.10.1
ip dhcp excluded-address 192.168.20.1

ip dhcp pool STAFF
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8

ip dhcp pool GUEST
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8

conf t
no ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/1
ip route 0.0.0.0 0.0.0.0 192.168.1.1

SW

vlan 10,20

interface Range f0/1 - 2  (พอร์ตที่เสียบไป Router และ AP)
 switchport mode trunk
 switchport trunk allowed vlan 10,20

AP 

conf t
  dot11 mbssid

dot11 ssid WIFI-STAFF
   vlan 10
   authentication open
   authentication key-management wpa version 2
   wpa-psk ascii 0 Staff@123
   mbssid guest-mode
exit

dot11 ssid WIFI-GUEST
   vlan 20
   authentication open
   authentication key-management wpa version 2
   wpa-psk ascii 0 Guest@123
   mbssid guest-mode
exit

interface Dot11Radio0
 no shutdown
 encryption vlan 10 mode ciphers aes-ccm
 encryption vlan 20 mode ciphers aes-ccm
 ssid WIFI-STAFF
 ssid WIFI-GUEST
exit

interface Dot11Radio1
 no shutdown
 encryption vlan 10 mode ciphers aes-ccm
 encryption vlan 20 mode ciphers aes-ccm
 ssid WIFI-STAFF
 ssid WIFI-GUEST

interface GigabitEthernet0
 no shutdown
interface GigabitEthernet0.10
 encapsulation dot1Q 10
 bridge-group 10
interface GigabitEthernet0.20
 encapsulation dot1Q 20
 bridge-group 20

interface Dot11Radio0.10
 encapsulation dot1Q 10
 bridge-group 10
interface Dot11Radio0.20
 encapsulation dot1Q 20
 bridge-group 20
interface GigabitEthernet0.10
 encapsulation dot1Q 10
 bridge-group 10
interface GigabitEthernet0.20
 encapsulation dot1Q 20
 bridge-group 20
   bridge-group 20
  exit
