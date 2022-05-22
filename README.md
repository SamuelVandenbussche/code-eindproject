#importeer libraries

import smbus

from time import sleep

import smtplib

from email.mime.multipart import MIMEMultipart

from email.mime.text import MIMEText

#setup

bmp_addr = 0x77

i2c = smbus.SMBus(1)


i2c.write_byte_data(bmp_addr, 0xf5, (5<<5))
i2c.write_byte_data(bmp_addr, 0xf4, ((5<<5) | (3<<0)))


dig_T1 = i2c.read_word_data(bmp_addr, 0x88)
dig_T2 = i2c.read_word_data(bmp_addr, 0x8A)
dig_T3 = i2c.read_word_data(bmp_addr, 0x8C)

if(dig_T2 > 32767):
    dig_T2 -= 65536
if(dig_T3 > 32767):
    dig_T3 -= 65536
    
#temperatuur calculeren

while True:

    d1 = i2c.read_byte_data(bmp_addr, 0xfa)
    d2 = i2c.read_byte_data(bmp_addr, 0xfb)
    d3 = i2c.read_byte_data(bmp_addr, 0xfc)

    adc_T = ((d1 << 16) | (d2 << 8) | d3) >> 4


    var1 = ((((adc_T>>3) - (dig_T1<<1))) * (dig_T2)) >> 11;
    var2 = (((((adc_T>>4) - (dig_T1)) * ((adc_T>>4) - (dig_T1))) >> 12) * (dig_T3)) >> 14;
    t_fine = var1 + var2;
    T = (t_fine * 5 + 128) >> 8;
    T = T / 100
    #mail sturen
    if(T < 15):
        mail_content = "Hello, the temperature is under 15 degrees celcius"
        #The mail addresses and password
        sender_address = 'r0879834.student@gmail.com'
        sender_pass = 'Plopkabouter123'
        receiver_address = 'karlientorney@hotmail.com'
        #Setup the MIME
        message = MIMEMultipart()
        message['From'] = sender_address
        message['To'] = receiver_address
        message['Subject'] = 'Reminder'   #The subject line
        #The body and the attachments for the mail
        message.attach(MIMEText(mail_content, 'plain'))
        #Create SMTP session for sending the mail
        session = smtplib.SMTP('smtp.gmail.com', 587) #use gmail with port
        session.starttls() #enable security
        session.login(sender_address, sender_pass) #login with mail_id and password
        text = message.as_string()
        session.sendmail(sender_address, receiver_address, text)
        session.quit()
        print('Mail Sent')
    print("Temperature: " +str(T))
    sleep(1)
