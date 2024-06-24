import sys
import requests
import json
from PyQt6.QtWidgets import (

    QApplication,
    QWidget,
    QVBoxLayout,
    QHBoxLayout,
    QLabel,
    QPushButton,
    QTextEdit,
    QMessageBox,
    QScrollArea,
    QFrame,
    QSpacerItem,
    QSizePolicy,
    QButtonGroup,
    QScrollBar,
)
from PyQt6.QtGui import QIcon, QPainter, QColor, QPixmap, QFont

from PyQt6.QtCore import Qt, QTimer, pyqtSlot, QThread, pyqtSignal


class ChatApp(QWidget):
    def __init__(self, parent=None):


        super().__init__(parent)
        self.api_url = "http://localhost:11434/api/generate"

        self.current_model = "qwen2:0.5b"


        self.conversation_history = ""
        self.init_ui()
        self.send_message(initial=True)



    def init_ui(self):
        self.setWindowTitle("Ollama V1.1")


        self.resize(600, 600)



        icon_pixmap = QPixmap(32, 32)


        icon_pixmap.fill(Qt.GlobalColor.transparent)


        painter = QPainter(icon_pixmap)
        painter.setRenderHint(QPainter.RenderHint.Antialiasing)



        painter.setBrush(QColor('white'))



        painter.setPen(Qt.PenStyle.NoPen)



        painter.drawEllipse(0, 0, 32, 32)




        painter.setPen(QColor('black'))

        font = QFont()
        font.setPointSize(14)
        painter.setFont(font)
        painter.drawText(icon_pixmap.rect(), Qt.AlignmentFlag.AlignCenter, "LG")


        painter.end()
        self.setWindowIcon(QIcon(icon_pixmap))



        self.output_browser = QScrollArea()
        self.output_browser.setWidgetResizable(True)


        self.chat_widget = QWidget()
        self.chat_layout = QVBoxLayout()
        self.chat_layout.addSpacerItem(QSpacerItem(20, 40, QSizePolicy.Policy.Minimum, QSizePolicy.Policy.Expanding))


        self.chat_widget.setLayout(self.chat_layout)


        self.output_browser.setWidget(self.chat_widget)




        self.input_text = CustomTextEdit()
        self.input_text.setFixedHeight(90)




        self.input_text.setVerticalScrollBarPolicy(Qt.ScrollBarPolicy.ScrollBarAsNeeded)

        self.input_text.setSizePolicy(QSizePolicy.Policy.Expanding, QSizePolicy.Policy.Fixed)

        self.input_text.returnPressed.connect(self.send_message)


        self.send_button = QPushButton("发送消息")

        self.send_button.setFixedHeight(40)


        self.clear_button = QPushButton("清除对话")

        self.clear_button.setFixedHeight(40)

        
        self.model_buttons = QButtonGroup(self)

        self.model_button_layout = QHBoxLayout()

        model_names = ["qwen2:0.5b", "qwen2:1.5b", "qwen2:7b"]

        for model_name in model_names:

            button = QPushButton(model_name)
            button.setCheckable(True)
            button.setFixedHeight(40)
            if model_name == self.current_model:

                button.setChecked(True)
            self.model_buttons.addButton(button)

            self.model_button_layout.addWidget(button)

        
        self.model_buttons.buttonClicked.connect(self.change_model)


        main_layout = QVBoxLayout()
        main_layout.addWidget(self.output_browser)


        input_layout = QVBoxLayout()
        input_layout.addWidget(self.input_text)


        button_layout = QHBoxLayout()
        button_layout.addLayout(self.model_button_layout)

        button_layout.addWidget(self.clear_button)

        button_layout.addWidget(self.send_button)

        button_layout.setSpacing(10)

        main_layout.addLayout(input_layout)
        main_layout.addLayout(button_layout)

        self.setLayout(main_layout)

        self.send_button.clicked.connect(self.send_message)

        self.clear_button.clicked.connect(self.clear_conversation)


    def send_message(self, initial=False):

        if initial:
            prompt = "你好"
        else:
            prompt = self.input_text.toPlainText().strip()
            if not prompt:
                return

        if prompt.lower() in ["退出", "exit"]:
            self.close()
            return

        if not initial:
            self.append_message(f"{prompt}", user=True)
            self.conversation_history += f"User: {prompt}\n"

        self.input_text.clear()
        self.thread = GenerateResponseThread(self.current_model, self.conversation_history, prompt, self.api_url)
        self.thread.response_generated.connect(self.handle_response)
        self.thread.start()

    def handle_response(self, response):
        if response:
            self.append_message_slow(f"Ollama: {response}", user=False)
            self.conversation_history += f"Ollama: {response}\n"
        else:
            self.show_message_box("错误", "未能获取有效的响应！")

    def change_model(self, button):
        selected_model = button.text()
        if selected_model != self.current_model:
            self.current_model = selected_model
            self.clear_conversation()
            self.send_message(initial=True)

    def clear_conversation(self):
        while self.chat_layout.count() > 1:
            item = self.chat_layout.takeAt(0)
            if item.widget():
                item.widget().deleteLater()
        self.conversation_history = ""
        self.append_message("对话已清除。", user=False)

    def append_message(self, message, user=False):
        message_label = QLabel(message)
        message_label.setWordWrap(True)
        message_label.setTextInteractionFlags(Qt.TextInteractionFlag.TextSelectableByMouse)
        message_frame = QFrame()
        message_layout = QHBoxLayout()

        if user:
            message_label.setStyleSheet("background-color: lightgreen; padding: 10px; border-radius: 10px;")
            message_layout.addSpacerItem(QSpacerItem(40, 20, QSizePolicy.Policy.Expanding, QSizePolicy.Policy.Minimum))
            message_layout.addWidget(message_label)
        else:
            message_label.setStyleSheet("background-color: lightblue; padding: 10px; border-radius: 10px;")
            message_layout.addWidget(message_label)
            message_layout.addSpacerItem(QSpacerItem(40, 20, QSizePolicy.Policy.Expanding, QSizePolicy.Policy.Minimum))

        message_frame.setLayout(message_layout)
        self.chat_layout.insertWidget(self.chat_layout.count() - 1, message_frame)
        QTimer.singleShot(100, lambda: self.output_browser.verticalScrollBar().setValue(self.output_browser.verticalScrollBar().maximum()))

    def append_message_slow(self, message, user=False):
        message_label = QLabel()
        message_label.setWordWrap(True)
        message_label.setTextInteractionFlags(Qt.TextInteractionFlag.TextSelectableByMouse)
        message_frame = QFrame()
        message_layout = QHBoxLayout()

        if user:
            message_label.setStyleSheet("background-color: lightgreen; padding: 10px; border-radius: 10px;")
            message_layout.addSpacerItem(QSpacerItem(40, 20, QSizePolicy.Policy.Expanding, QSizePolicy.Policy.Minimum))
            message_layout.addWidget(message_label)
        else:
            message_label.setStyleSheet("background-color: lightblue; padding: 10px; border-radius: 10px;")
            message_layout.addWidget(message_label)
            message_layout.addSpacerItem(QSpacerItem(40, 20, QSizePolicy.Policy.Expanding, QSizePolicy.Policy.Minimum))

        message_frame.setLayout(message_layout)
        self.chat_layout.insertWidget(self.chat_layout.count() - 1, message_frame)
        QTimer.singleShot(100, lambda: self.output_browser.verticalScrollBar().setValue(self.output_browser.verticalScrollBar().maximum()))

        self.show_message_slowly(message_label, message)

    def show_message_slowly(self, label, message):
        self.current_char = 0
        self.message = message
        self.label = label

        self.timer = QTimer()
        self.timer.timeout.connect(self.update_message)
        self.timer.start(50)

    @pyqtSlot()
    def update_message(self):
        self.current_char += 1
        self.label.setText(self.message[:self.current_char])

        if self.current_char >= len(self.message):
            self.timer.stop()
            QTimer.singleShot(100, lambda: self.output_browser.verticalScrollBar().setValue(self.output_browser.verticalScrollBar().maximum()))

    def show_message_box(self, title, message):
        msg_box = QMessageBox()
        msg_box.setWindowTitle(title)
        msg_box.setText(message)
        msg_box.exec()

class GenerateResponseThread(QThread):
    response_generated = pyqtSignal(str)

    def __init__(self, model, conversation_history, prompt, api_url):

        super().__init__()
        self.model = model
        self.conversation_history = conversation_history
        self.prompt = prompt
        self.api_url = api_url

    def run(self):
        headers = {
            'Content-Type': 'application/json'


        }
        payload = {
            "model": self.model,
            "prompt": self.conversation_history + f"User: {self.prompt}"


        }

        try:
            response = requests.post(self.api_url, headers=headers, data=json.dumps(payload))


            response.raise_for_status()

            json_objects = response.text.splitlines()



            full_response = ""
            for json_object in json_objects:


                try:
                    response_data = json.loads(json_object)

                    full_response += response_data.get('response', '')


                except json.JSONDecodeError as e:


                    print(f"解析JSON对象失败: {e}")



            self.response_generated.emit(full_response)


        except requests.exceptions.RequestException as e:


            print(f"HTTP请求失败: {e}")


            self.response_generated.emit("")

class CustomTextEdit(QTextEdit):
    returnPressed = pyqtSignal()

    def __init__(self, parent=None):


        super().__init__(parent)

    def keyPressEvent(self, event):



        if event.key() == Qt.Key.Key_Return and not event.modifiers() & Qt.KeyboardModifier.ShiftModifier:



            self.returnPressed.emit()
        else:
            super().keyPressEvent(event)

if __name__ == "__main__":
    app = QApplication(sys.argv)



    chat_app = ChatApp()
    chat_app.show()
    sys.exit(app.exec())




