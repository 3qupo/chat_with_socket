
import sys
import os
import rsa
import time
import socket
import shelve
import threading
from PyQt5 import QtCore
from PyQt5 import QtGui
from PyQt5 import QtWidgets
from des import *

# Мониторинг входящих сообщений
class message_monitor(QtCore.QThread):
    mysignal = QtCore.pyqtSignal(str)

    def __init__(self, server_socket, private_key, parent = None):

        QtCore.QThread.__init__(self, parent)
        self.server_socket = server_socket
        self.private_key = private_key
        self.message = None

    def run(self):
        while True:
            try:
                # Зашифрованные данные от собеседника
                self.message = self.server_socket.recv(1024)
                decrypt_message = rsa.decrypt(self.message, self.private_key)
                self.mysignal.emit(decrypt_message.decode('utf-8'))

            except:
                # Не зашифрованные данные от сервера
                self.mysignal.emit(self.message.decode('utf-8'))

class Client(QtWidgets.QMainWindow):
    def __init__(self, parent = None):

        QtWidgets.QWidget.__init__(self, parent)
        self.ui = Ui_MainWindow()
        self.ui.setupUi(self)

        self.ip = None
        self.port = None
        self.friend_public_key = None

        # Ключи шифрования клиента
        self.mypublickey = None
        self.myprivatekey = None

        # Проверка на наличие идентификатора собеседника
        if len(os.listdir('friend_id')) == 0:
            self.ui.lineEdit.setEnabled(False)
            self.ui.pushButton.setEnabled(False)
            self.ui.pushButton_2.setEnabled(False)
            self.ui.pushButton_3.setEnabled(False)
            message = 'Поместите идентификатор собеседника в файл "friend_id"'
            self.ui.plainTextEdit.appendPlainText(message)

        # Проверка на создание личного идентификатора
        if not os.path.exists('private'):
            self.ui.lineEdit.setEnabled(False)
            self.ui.pushButton.setEnabled(False)
            self.ui.pushButton_2.setEnabled(False)
            self.ui.pushButton_3.setEnabled(False)
            message = 'Также необходимо сгенерировать свой идентификатор'
            self.ui.plainTextEdit.appendPlainText(message)

        else:
            # Подгружаем данные текущего клиента
            with shelve.open('private') as file:
                self.mypublickey = file['public']
                self.myprivatekey = file['private']
                self.ip = file['ip']
                self.port = file['port']

            # Загрузка данных собеседника
            with shelve.open(os.path.join('friend_id', os.listdir('friend_id')[0])) as file:
                self.friend_public_key = file['publickey']

            message = 'Подключитесь к серверу'
            self.ui.plainTextEdit.appendPlainText(message)
            self.ui.lineEdit.setEnabled(False)
            self.ui.pushButton.setEnabled(False)
            self.ui.pushButton_2.setEnabled(False)
            self.ui.pushButton_3.setEnabled(True)

        # Обработка кнопок
        self.ui.pushButton.clicked.connect(self.send_message)
        self.ui.pushButton_2.clicked.connect(self.connect_server)
        self.ui.pushButton_3.clicked.connect(self.clear_panel)


    # Подключение к серверу
    def connect_server(self):
        try:

            self.tcp_client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            self.tcp_client.connect((self.ip, self.port)); time.sleep(2)

            # Запуск мониторинга входящих сообщений
            self.message_monitor = message_monitor(self.tcp_client, self.myprivatekey)
            self.message_monitor.mysignal.connect(self.update_chat)
            self.message_monitor.start()

            # Действия с объектами
            self.ui.LineEdit.setEnabled(False)
            self.ui.pushButton.setEnabled(True)
            self.ui.plainTextEdit.setEnabled(True)
            self.ui.pushButton_2.setEnabled(False)
            self.ui.pushButton_3.setEnabled(False)

        except:
            self.ui.plainTextEdit.clear()
            self.ui.plainTextEdit.appendPlainText("Ошибка подключения к серверу!")
            self.ui.plainTextEdit.appendPlainText("Измените идентификаторы и повторите попытку")

    # Отправка сообщений
    def send_message(self):
        try:

            if len(self.ui.lineEdit.text()) > 0:
                message = self.ui.lineEdit.text()
                crypto_message = rsa.encrypt(message.encode('utf-8'), self.friend_public_key)

                self.ui.plainTextEdit.appendPlainText(f'[Вы]: {message}')
                self.tcp_client.send(crypto_message)
                self.ui.lineEdit.clear()

        except:
            sys.exit()

    # Генерация ключей шифрования
    def generate_encrypt(self):
        if len(self.ui.lineEdit.text()) > 0: # Еще одно условие? Надо доделать, перед этим поменять lineedit
            pubkey, privkey = rsa.newkeys(512)

            with shelve.open('your_id') as file:
                file['pubkey'] = pubkey
                file['ip'] = str(self.ui.plainTextEdit.text()) # Тут один
                file['port'] = str(self.ui.plainTextEdit.text()) # Тут другой

            with shelve.open('private') as file:
                file['pubkey'] = pubkey
                file['privkey'] = privkey
                file['ip'] = str(self.ui.plainTextEdit.text())  # Тут один
                file['port'] = str(self.ui.plainTextEdit.text())  # Тут другой

            self.ui.plainTextEdit.appendPlainText('Создан "your_id" идентификатор')
            self.ui.plainTextEdit.appendPlainText('Передайте его собеседнику и сможете начать диалог')

        else:
            self.ui.plainTextEdit.clear()
            self.ui.plainTextEdit.appendPlainText("Проверьте правильность вводимых данных")

    '''        
    else:
        self.ui.plainTextEdit.clear()
        self.ui.plainTextEdit.appendPlainText("Проверьте правильность вводимых данных")
    '''

    # Закрытие соединения
    def closeEvent(self, event):
        try:
            self.tcp_client.send(b'exit')
            self.tcp_client.close()

        except:
            pass

    # Обновление диалогового окна
    def update_chat(self, value):
        self.ui.plainTextEdit.appendHtml(value)

    # Очищение диалогового окна
    def clear_panel(self):
        self.ui.plainTextEdit.clear()

if __name__ == "main":
    app = QtWidgets.QApplication(sys.argv)
    myapp = Client()
    myapp.show()
    sys.exit(app.exec_())
