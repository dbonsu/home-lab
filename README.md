# Homelab Journey

The idea behind this repo is to document my home lab journey. There are a few things I want to accomplish with this home lab. In no order:

- experiment
- self hosting
- media library, cuz everyone loves to look back at some old pics


Current topology: I expect this to change a lot in the beginning.

        opnsense 
        (10.0.x.1)
           |
           |
        switch
       (10.0.x.2)
       |        |       |       |       | 
       Main     Guest    VPN    Svc     AP
       |        |       |       |       |
       |        |       |       |       |
       |        |       |       |       |Port 1(T), Port 6(U)
       |        |       |       |Port 1(T), Port 5(U)
       |        |       |Port 1(T), Port 4(U)       
       |        Port 1(T), Port 3(U)
       Port 1(T), Port 2(U)
       |
       |
       AP 
       (10.0.x.3)