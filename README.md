# Лабораторная работа №12

## Задание

Разработать графический интерфейс для просмотра картотеки Spotify

_Вы можете отходить от дизайна, но должен присутствовать обязательный функционал_

### Обязательный функционал

- Поиск по названию песни, исполнителю, альбому
- Сортировка по дате добавления (дефолт), длительности, популярности
- Информация должна забираться с пути `http://omsktec-playgrounds.ru/algos/lab13`

_Если у вас сайт заблокирован Ростелекомом, то забирайте с IP адреса (`http://147.45.158.220/algos/lab13`)_

- Для запросов должна быть использована библиотека `requests` (не забудьте установить её)

## Референс

<img src="./.repo/finished.png" />

## Работа с таблицами

Компонент Treeview в Tkinter используется для отображения табличных данных, а также для создания иерархических структур (деревьев).

### Создание таблицы

```python
import tkinter as tk
from tkinter import ttk

# Создание окна
root = tk.Tk()
root.title("Пример таблицы")

# Создание таблицы с указанием колонок
columns = ('id', 'name', 'age')
table = ttk.Treeview(root, columns=columns, show='headings')

# Размещение таблицы
table.pack(fill=tk.BOTH, expand=True)
```

Параметр `show='headings'` означает, что будут отображаться только заголовки колонок без дополнительной колонки для структуры дерева.

### Настройка колонок

```python
# Настройка заголовков
table.heading('id', text='ID')
table.heading('name', text='Имя')
table.heading('age', text='Возраст')

# Настройка ширины колонок
table.column('id', width=50, anchor=tk.CENTER)
table.column('name', width=150)
table.column('age', width=70, anchor=tk.CENTER)
```

### Добавление данных

```python
# Добавление данных в таблицу
data = [
    (1, 'Иван', 25),
    (2, 'Мария', 22),
    (3, 'Алексей', 30)
]

for item in data:
    table.insert('', tk.END, values=item)
```

Первый параметр метода `insert()` - это родительский элемент (пустая строка для корневого уровня), второй параметр - индекс (tk.END для добавления в конец), а `values` - кортеж со значениями для колонок.

### Обработка выделения строк

```python
def on_item_select(event):
    # Получение выделенных элементов
    selected_items = table.selection()
    if selected_items:  # Проверка, что что-то выбрано
        item = selected_items[0]  # Берем первый выбранный элемент
        values = table.item(item, 'values')
        print(f"Выбран: {values}")

# Привязка события выбора элемента
table.bind('<<TreeviewSelect>>', on_item_select)
```

### Сортировка и фильтрация

Treeview не имеет встроенных функций сортировки и фильтрации, но вы можете их реализовать самостоятельно:

```python
def sort_by_column(tree, col, reverse):
    # Получение данных из таблицы
    data = [(tree.item(item, 'values'), item) for item in tree.get_children('')]
    
    # Сортировка данных
    data.sort(key=lambda x: x[0][columns.index(col)], reverse=reverse)
    
    # Перестановка элементов
    for index, (values, item) in enumerate(data):
        tree.move(item, '', index)
    
    # Следующий вызов метода изменит направление сортировки
    tree.heading(col, command=lambda: sort_by_column(tree, col, not reverse))

# Привязка сортировки к заголовкам колонок
for col in columns:
    table.heading(col, command=lambda c=col: sort_by_column(table, c, False))
```

## Работа с картинками

Для работы с изображениями в Tkinter рекомендуется использовать библиотеку PIL/Pillow.

### Установка Pillow

```
pip install pillow
```

### Загрузка изображений

```python
from PIL import Image, ImageTk
import requests
from io import BytesIO

# Загрузка изображения из файла
image = Image.open('image.jpg')

# Загрузка изображения из URL
def load_image_from_url(url):
    response = requests.get(url)
    if response.status_code == 200:
        return Image.open(BytesIO(response.content))
    return None
```

### Изменение размера изображений

```python
# Изменение размера с сохранением пропорций
def resize_image(image, max_width, max_height):
    # Получаем исходные размеры
    width, height = image.size
    
    # Находим коэффициент масштабирования
    ratio = min(max_width / width, max_height / height)
    
    # Новые размеры
    new_width = int(width * ratio)
    new_height = int(height * ratio)
    
    # Изменяем размер
    return image.resize((new_width, new_height), Image.LANCZOS)
```

### Отображение изображений в Tkinter

```python
# Преобразование изображения PIL в формат Tkinter
photo_image = ImageTk.PhotoImage(image)

# Создание метки для отображения
image_label = ttk.Label(root)
image_label.pack()

# Отображение изображения
image_label.config(image=photo_image)

# Сохраняем ссылку на изображение, чтобы избежать проблем со сборщиком мусора
image_label.image = photo_image
```

### Асинхронная загрузка изображений

Чтобы избежать "зависания" интерфейса при загрузке изображений, особенно из сети, используйте потоки:

```python
import threading

def download_and_display_image(url, label):
    def download():
        try:
            # Загрузка в отдельном потоке
            response = requests.get(url)
            if response.status_code == 200:
                image = Image.open(BytesIO(response.content))
                image = resize_image(image, 300, 300)
                photo = ImageTk.PhotoImage(image)
                
                # Обновление UI нужно делать в главном потоке
                label.after(0, lambda: update_label(label, photo))
        except Exception as e:
            print(f"Ошибка загрузки изображения: {e}")
    
    def update_label(label, photo):
        label.config(image=photo)
        label.image = photo  # Сохраняем ссылку
    
    # Запуск загрузки в отдельном потоке
    threading.Thread(target=download).start()
```
import tkinter as tk
from tkinter import ttk
import requests
from PIL import Image, ImageTk
from io import BytesIO
import threading
from datetime import datetime

class SpotifyApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Spotify Картотека")
        self.root.geometry("1000x600")
        
        # Данные
        self.data = []
        self.filtered_data = []
        self.current_sort = "date_added"
        self.sort_reverse = False
        
        # Создание интерфейса
        self.create_widgets()
        
        # Загрузка данных
        self.load_data()
    
    def create_widgets(self):
        # Панель поиска
        search_frame = ttk.Frame(self.root, padding="10")
        search_frame.pack(fill=tk.X)
        
        ttk.Label(search_frame, text="Поиск:").pack(side=tk.LEFT)
        
        self.search_var = tk.StringVar()
        self.search_entry = ttk.Entry(search_frame, textvariable=self.search_var, width=40)
        self.search_entry.pack(side=tk.LEFT, padx=5)
        self.search_entry.bind("<KeyRelease>", self.apply_filters)
        
        # Кнопки сортировки
        sort_frame = ttk.Frame(self.root, padding="10")
        sort_frame.pack(fill=tk.X)
        
        ttk.Label(sort_frame, text="Сортировка:").pack(side=tk.LEFT)
        
        self.sort_buttons = {
            "date_added": ttk.Button(sort_frame, text="По дате добавления", 
                                   command=lambda: self.sort_data("date_added")),
            "duration": ttk.Button(sort_frame, text="По длительности", 
                                 command=lambda: self.sort_data("duration")),
            "popularity": ttk.Button(sort_frame, text="По популярности", 
                                   command=lambda: self.sort_data("popularity"))
        }
        
        for btn in self.sort_buttons.values():
            btn.pack(side=tk.LEFT, padx=5)
        
        # Таблица с данными
        self.tree_frame = ttk.Frame(self.root)
        self.tree_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=(0, 10))
        
        columns = ("track", "artist", "album", "duration", "popularity", "date_added")
        self.tree = ttk.Treeview(self.tree_frame, columns=columns, show="headings", selectmode="browse")
        
        # Настройка колонок
        self.tree.heading("track", text="Трек", command=lambda: self.sort_treeview("track"))
        self.tree.heading("artist", text="Исполнитель", command=lambda: self.sort_treeview("artist"))
        self.tree.heading("album", text="Альбом", command=lambda: self.sort_treeview("album"))
        self.tree.heading("duration", text="Длительность", command=lambda: self.sort_treeview("duration"))
        self.tree.heading("popularity", text="Популярность", command=lambda: self.sort_treeview("popularity"))
        self.tree.heading("date_added", text="Дата добавления", command=lambda: self.sort_treeview("date_added"))
        
        self.tree.column("track", width=200)
        self.tree.column("artist", width=150)
        self.tree.column("album", width=200)
        self.tree.column("duration", width=80, anchor="center")
        self.tree.column("popularity", width=80, anchor="center")
        self.tree.column("date_added", width=120, anchor="center")
        
        # Полоса прокрутки
        scrollbar = ttk.Scrollbar(self.tree_frame, orient="vertical", command=self.tree.yview)
        self.tree.configure(yscrollcommand=scrollbar.set)
        scrollbar.pack(side="right", fill="y")
        self.tree.pack(side="left", fill="both", expand=True)
        
        # Детальная информация
        self.detail_frame = ttk.Frame(self.root, padding="10")
        self.detail_frame.pack(fill=tk.X)
        
        self.album_art = ttk.Label(self.detail_frame)
        self.album_art.pack(side="left", padx=10)
        
        self.detail_text = tk.Text(self.detail_frame, height=5, width=60, state="disabled")
        self.detail_text.pack(side="left", fill=tk.BOTH, expand=True)
        
        # Привязка события выбора
        self.tree.bind("<<TreeviewSelect>>", self.on_item_select)
    
    def load_data(self):
        try:
            # Загрузка данных с сервера
            response = requests.get("http://omsktec-playgrounds.ru/algos/1ab13")
            if response.status_code == 200:
                self.data = response.json()
                self.filtered_data = self.data.copy()
                self.sort_data("date_added")
            else:
                # Попробуем альтернативный URL
                response = requests.get("http://147.45.158.220/algos/Lab13")
                if response.status_code == 200:
                    self.data = response.json()
                    self.filtered_data = self.data.copy()
                    self.sort_data("date_added")
                else:
                    raise Exception("Не удалось загрузить данные")
        except Exception as e:
            print(f"Ошибка загрузки данных: {e}")
            # Создаем тестовые данные для демонстрации
            self.data = [
                {
                    "track": "Test Track",
                    "artist": "Test Artist",
                    "album": "Test Album",
                    "duration": "3:45",
                    "popularity": 50,
                    "date_added": "2023-01-01",
                    "album_art": "https://via.placeholder.com/150"
                }
            ]
            self.filtered_data = self.data.copy()
            self.sort_data("date_added")
    
    def apply_filters(self, event=None):
        search_term = self.search_var.get().lower()
        
        if not search_term:
            self.filtered_data = self.data.copy()
        else:
            self.filtered_data = [
                item for item in self.data
                if (search_term in item["track"].lower() or 
                    search_term in item["artist"].lower() or 
                    search_term in item["album"].lower())
            ]
        
        self.sort_data(self.current_sort)
    
    def sort_data(self, sort_key):
        self.current_sort = sort_key
        
        # Сбросим выделение всех кнопок сортировки
        for btn in self.sort_buttons.values():
            btn.state(["!pressed"])
        
        # Подсветим активную кнопку сортировки
        if sort_key == "date_added":
            self.sort_buttons["date_added"].state(["pressed"])
        elif sort_key == "duration":
            self.sort_buttons["duration"].state(["pressed"])
        elif sort_key == "popularity":
            self.sort_buttons["popularity"].state(["pressed"])
        
        # Сортировка данных
        if sort_key == "date_added":
            self.filtered_data.sort(key=lambda x: datetime.strptime(x[sort_key], "%Y-%m-%d"), reverse=self.sort_reverse)
        elif sort_key == "duration":
            self.filtered_data.sort(key=lambda x: self.duration_to_seconds(x[sort_key]), reverse=self.sort_reverse)
        else:
            self.filtered_data.sort(key=lambda x: x[sort_key], reverse=self.sort_reverse)
        
        self.update_treeview()
    
    def sort_treeview(self, column):
        # Простая сортировка по колонкам таблицы
        items = [(self.tree.set(item, column), item) for item in self.tree.get_children("")]
        items.sort(reverse=self.sort_reverse)
        
        for index, (val, item) in enumerate(items):
            self.tree.move(item, "", index)
        
        self.sort_reverse = not self.sort_reverse
    
    def duration_to_seconds(self, duration_str):
        try:
            minutes, seconds = map(int, duration_str.split(":"))
            return minutes * 60 + seconds
        except:
            return 0
    
    def update_treeview(self):
        # Очистка таблицы
        for item in self.tree.get_children():
            self.tree.delete(item)
        
        # Заполнение таблицы
        for item in self.filtered_data:
            self.tree.insert("", "end", values=(
                item["track"],
                item["artist"],
                item["album"],
                item["duration"],
                item["popularity"],
                item["date_added"]
            ))
    
    def on_item_select(self, event):
        selected_item = self.tree.focus()
        if not selected_item:
            return
        
        item_data = self.tree.item(selected_item)
        values = item_data["values"]
        
        # Находим полные данные о выбранном треке
        track_info = next(
            (item for item in self.filtered_data 
             if (item["track"] == values[0] and 
                 item["artist"] == values[1] and 
                 item["album"] == values[2])),
            None
        )
        
        if track_info:
            # Обновляем детальную информацию
            self.detail_text.config(state="normal")
            self.detail_text.delete(1.0, tk.END)
            
            info_text = (
                f"Трек: {track_info['track']}\n"
                f"Исполнитель: {track_info['artist']}\n"
                f"Альбом: {track_info['album']}\n"
                f"Длительность: {track_info['duration']}\n"
                f"Популярность: {track_info['popularity']}\n"
                f"Дата добавления: {track_info['date_added']}"
            )
            
            self.detail_text.insert(tk.END, info_text)
            self.detail_text.config(state="disabled")
            
            # Загрузка обложки альбома
            self.load_album_art(track_info.get("album_art", ""))
    
    def load_album_art(self, url):
        if not url:
            # Очищаем изображение, если URL нет
            self.album_art.config(image="")
            self.album_art.image = None
            return
        
        # Загрузка изображения в отдельном потоке
        threading.Thread(target=self._download_image, args=(url,), daemon=True).start()
    
    def _download_image(self, url):
        try:
            response = requests.get(url, stream=True)
            if response.status_code == 200:
                image_data = response.content
                image = Image.open(BytesIO(image_data))
                image = image.resize((150, 150), Image.LANCZOS)
                photo = ImageTk.PhotoImage(image)
                
                # Обновляем UI в главном потоке
                self.root.after(0, lambda: self._update_image(photo))
        except Exception as e:
            print(f"Ошибка загрузки изображения: {e}")
    
    def _update_image(self, photo_image):
        self.album_art.config(image=photo_image)
        self.album_art.image = photo_image  # Сохраняем ссылку


if __name__ == "__main__":
    root = tk.Tk()
    app = SpotifyApp(root)
    root.mainloop()
