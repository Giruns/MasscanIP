# MasscanIP
from netaddr import IPNetwork
import os,sys,time,socket,colorama
from threading import *
from threading import Thread
try:
    from queue import Queue
except:
    from Queue import Queue
colorama.init()

class Worker(Thread):
    def __init__(self, tasks):
        Thread.__init__(self)
        self.tasks = tasks
        self.daemon = True
        self.start()

    def run(self):
        while True:
            func, args, kargs = self.tasks.get()
            try:
                func(*args, **kargs)
            except Exception as e:
                print(e)
            self.tasks.task_done()


class ThreadPool:
    def __init__(self, num_threads):
        self.tasks = Queue(num_threads)
        for _ in range(num_threads): Worker(self.tasks)

    def add_task(self, func, *args, **kargs):
        self.tasks.put((func, args, kargs))

    def wait_completion(self):
        self.tasks.join()
def cek_port(txt):
    try:
        sc = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
        sc.settimeout(0.1)
        so = sc.connect_ex((txt, 80))
        sc.close()
        if str(so) == '0':
            print(f'\033[92m{txt}\033[0m')
            open('IP_LIVE.txt','a').write(txt+'\n')
        else:
            sc = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
            sc.settimeout(0.1)
            so = sc.connect_ex((txt, 443))
            sc.close()
            if str(so) == '0':
                print(f'\033[92m{txt}\033[0m')
                open('IP_LIVE.txt','a').write(txt+'\n')
            else:
                print(f'\033[91m{txt}\033[0m')
    except:
        print(f'\033[91m{txt}\033[0m')
ipe = input('[+] List IP CIDR : ')
th = ThreadPool(10) # Kecepatan Scanning
for myip in open(ipe,'r').read().splitlines():
    try:
        for x in IPNetwork(myip):
            th.add_task(cek_port,str(x))
        th.wait_completion()
    except Exception as e:
        print(e)
