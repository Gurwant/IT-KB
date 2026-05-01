# SPF

DNS Field to validate if the IP adress from where the email was send is legitimate

---

Email received from Linkedin : 
<img width="390" height="26" alt="image" src="https://github.com/user-attachments/assets/54ea3f31-e743-455f-bd05-f49afc6a1b1a" />


Check the sender details :
<img width="660" height="29" alt="image" src="https://github.com/user-attachments/assets/38fa30f4-1f38-4e17-81c5-17cbdd399a8b" />

Check the SPF for linkedin.com : run *dig linkedin.com TXT*
We find the following : <img width="1909" height="60" alt="image" src="https://github.com/user-attachments/assets/95fc159a-eb08-4f20-90bd-9b2490499376" />
We can see the subnet 108.174.0.0/24 that does validate the adress seen in the sender details : 108.174.0.155
