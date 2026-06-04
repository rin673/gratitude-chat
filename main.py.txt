import flet as ft
import json

def main(page: ft.Page):
    page.title = "部署チャット"
    page.scroll = "adaptive"

    departments = [
        {"name": "営業部", "color": "#f7c6c7"},
        {"name": "開発部", "color": "#a8d5ba"},
        {"name": "総務部", "color": "#cdeccd"},
        {"name": "人事部", "color": "#fdf6ec"},
    ]

    chats = {d["name"]: ft.Column(spacing=10, expand=True) for d in departments}

    bg_colors = {
        "営業部": "#fff0f3",
        "開発部": "#e3f2fd",
        "総務部": "#e8f5e9",
        "人事部": "#fdf6ec",
    }

    def load_chat_history():
        try:
            with open("chat_history.json", "r", encoding="utf-8") as f:
                data = json.load(f)
            for dept, messages in data.items():
                for msg in messages:
                    chats[dept].controls.append(
                        ft.Container(
                            content=ft.Text(msg, color="#333333"),
                            bgcolor="#cdeccd",
                            border_radius=10,
                            padding=10,
                            width=250,
                        )
                    )
        except FileNotFoundError:
            pass

    def save_chat_history():
        data = {dept: [msg.content.value for msg in chat.controls] for dept, chat in chats.items()}
        with open("chat_history.json", "w", encoding="utf-8") as f:
            json.dump(data, f, ensure_ascii=False, indent=2)

    def show_department_list():
        page.clean()
        page.bgcolor = "#f8f9fa"
        page.add(
            ft.Column(
                controls=[
                    ft.Text("部署チャット一覧", size=22, weight="bold"),
                    *[
                        ft.Container(
                            content=ft.Row(
                                controls=[
                                    ft.Text(f"💬 {d['name']}", size=18, weight="bold"),
                                    ft.Text("最新メッセージあり", color="#555555"),
                                ],
                                alignment=ft.MainAxisAlignment.SPACE_BETWEEN,
                            ),
                            bgcolor=d["color"],
                            padding=10,
                            border_radius=10,
                            on_click=lambda e, name=d["name"]: open_chat(name),
                        )
                        for d in departments
                    ],
                    ft.Text("どの部署にも入って感謝を送れます（匿名）", color="#777777"),
                ],
                spacing=10,
            )
        )

    def open_chat(dept_name):
        page.clean()
        page.bgcolor = bg_colors.get(dept_name, "#f8f9fa")
        chat_area = chats[dept_name]
        input_field = ft.TextField(hint_text="メッセージを入力", expand=True)
        send_button = ft.ElevatedButton(
            text="送信",
            bgcolor="#a8d5ba",
            color="#ffffff",
            on_click=lambda e: send_message(dept_name, input_field, chat_area),
        )

        page.add(
            ft.Column(
                controls=[
                    ft.Row(
                        controls=[
                            ft.IconButton(ft.icons.ARROW_BACK, on_click=lambda e: show_department_list()),
                            ft.Text(f"{dept_name} チャット", size=20, weight="bold"),
                        ]
                    ),
                    ft.Container(content=chat_area, expand=True),
                    ft.Row(controls=[input_field, send_button]),
                ],
                expand=True,
            )
        )

    def send_message(dept_name, input_field, chat_area):
        if input_field.value.strip():
            chat_area.controls.append(
                ft.Container(
                    content=ft.Text(input_field.value, color="#333333"),
                    bgcolor="#cdeccd",
                    border_radius=10,
                    padding=10,
                    width=250,
                )
            )
            input_field.value = ""
            save_chat_history()
            page.update()

    load_chat_history()
    show_department_list()

ft.app(target=main)
