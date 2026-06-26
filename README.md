from flask import Flask, render_template_string, request

app = Flask(__name__)

# База данных курсов
COURSES_DATA = {
    "python": {
        "title": "🐍 Курс по Python",
        "description": "Изучи самый популярный язык для нейросетей, сайтов и автоматизации.",
        "video_url": "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ForBiggerBlazes.mp4",
        "lessons": [
            {"id": 1, "title": "1. Введение в Python и синтаксис", "content": "Python славится своим простым синтаксисом. Здесь не нужны точки с запятой, а блоки кода выделяются отступами. Твой первый код: print('Привет, мир!')"},
            {"id": 2, "title": "2. Переменные и типы данных", "content": "Переменные хранят данные. В Python не нужно объявлять тип заранее: x = 10 (это целое число), name = 'Иван' (это строка)."},
            {"id": 3, "title": "3. Условия и логика (if/elif/else)", "content": "Позволяют программе принимать решения. Пример: if age >= 18: print('Доступ разрешен')."}
        ],
        "quiz": {
            "question": "Как правильно создать переменную со значением 5 в Python?",
            "options": ["int x = 5;", "x = 5", "new var x = 5"],
            "correct": "x = 5",
            "explain": "В Python не нужно писать ключевые слова перед переменной и ставить точку с запятой!"
        }
    },
    "javascript": {
        "title": "🌐 Курс по JavaScript",
        "description": "Оживи свои веб-страницы. Язык для создания интерактивных сайтов и фронтенда.",
        "video_url": "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4",
        "lessons": [
            {"id": 1, "title": "1. Что такое JavaScript и где он работает?", "content": "JavaScript (JS) выполняется прямо в браузере. Он позволяет делать интерактивные кнопки, анимации и обрабатывать клики пользователей."},
            {"id": 2, "title": "2. Переменные (let, const, var)", "content": "В современном JS для создания переменных используются let (для изменяемых значений) и const (для констант). Например: let score = 0;."},
            {"id": 3, "title": "3. Работа с функциями", "content": "Функции в JS объявляются через слово function: function sayHi() { alert('Привет!'); }."}
        ],
        "quiz": {
            "question": "Какое ключевое слово создает переменную, которую НЕЛЬЗЯ изменить?",
            "options": ["let", "var", "const"],
            "correct": "const",
            "explain": "Ключевое слово const означает 'константа' — значение этой переменной перезаписать нельзя."
        }
    },
    "cpp": {
        "title": "💻 Курс по C++",
        "description": "Мощный язык для создания крутых 3D-игр, операционных систем и быстрых программ.",
        "video_url": "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ForBiggerEscapes.mp4",
        "lessons": [
            {"id": 1, "title": "1. Основы C++ и компиляция", "content": "C++ — это компилируемый язык. Программа жестко проверяется перед запуском. Базовый каркас требует подключения библиотеки: #include <iostream>."},
            {"id": 2, "title": "2. Строгая типизация", "content": "Здесь ты обязан говорить компьютеру, что именно лежит в переменной: int number = 100; или float pi = 3.14;."},
            {"id": 3, "title": "3. Точки с запятой и функция main", "content": "Весь код в C++ пишется внутри главной функции int main() { ... }, а в конце каждой строчки обязательно ставится ;."}
        ],
        "quiz": {
            "question": "Что обязательно нужно ставить в конце почти каждой строки кода в C++?",
            "options": ["Точку с запятой (;)", "Двоеточие (:)", "Ничего не нужно"],
            "correct": "Точку с запятой (;)",
            "explain": "Без точки с запятой компилятор C++ выдаст ошибку и программа не запустится."
        }
    }
}

MOBILE_STYLES = """
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
    body { font-family: 'Segoe UI', Arial, sans-serif; background-color: #121212; color: #e0e0e0; padding: 20px; margin: 0; }
    h1 { color: #00ffcc; font-size: 30px; text-align: center; margin-bottom: 5px; }
    p { font-size: 18px; color: #b0b0b0; }
    .container { max-width: 600px; margin: 0 auto; }
    .course-card { background: #1e1e1e; border: 2px solid #333; border-radius: 16px; padding: 20px; margin-bottom: 20px; border-left: 6px solid #00ffcc; }
    .english-card { border-left-color: #ff3366; }
    .course-title { font-size: 24px; color: #fff; margin: 0 0 10px 0; font-weight: bold; }
    .btn { display: block; background-color: #252525; color: #00ffcc; padding: 20px; text-decoration: none; border-radius: 16px; font-weight: bold; font-size: 22px; text-align: center; border: 1px solid #333; margin-top: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.4); }
    .english-btn { color: #ff3366; border-color: #442233; }
    .btn:active { background-color: #333; }
    .back-btn { display: inline-block; margin-top: 25px; color: #ff3366; font-size: 18px; text-decoration: none; font-weight: bold; }
    video { width: 100%; border-radius: 12px; margin-top: 10px; background: #000; border: 1px solid #333; }
    .option { background: #2a2a2a; padding: 16px; border-radius: 12px; margin-bottom: 12px; font-size: 18px; display: flex; align-items: center; }
    .option input { margin-right: 15px; transform: scale(1.4); }
    .submit-btn { background: #00ffcc; color: #121212; border: none; padding: 16px; width: 100%; border-radius: 12px; font-size: 20px; font-weight: bold; cursor: pointer; }
    .english-submit { background: #ff3366; color: white; }
</style>
"""

# 1. ГЛАВНАЯ СТРАНИЦА
@app.route('/')
def home():
    html_content = f"""
    <html>
    <head>{MOBILE_STYLES}</head>
    <body>
        <div class="container" style="margin-top: 40px;">
            <h1>Платформа Саморазвития 🚀</h1>
            <p style="text-align:center; font-size:18px; margin-bottom:40px;">Прокачай свои мозги на максимум</p>
            
            <a class="btn" href="/coding_hub" style="padding: 30px 20px; border-left: 6px solid #00ffcc;">
                🔥 Научись программировать,<br>не давай себя унизить
            </a>
            
            <a class="btn english-btn" href="/english" style="padding: 25px 20px; margin-top: 25px; border-left: 6px solid #ff3366;">
                🇬🇧 Английский для IT
            </a>
        </div>
    </body>
    </html>
    """
    return render_template_string(html_content)

# 2. ХАБ ПРОГРАММИРОВАНИЯ
@app.route('/coding_hub')
def coding_hub():
    cards_html = ""
    for lang_key, lang_data in COURSES_DATA.items():
        cards_html += f"""
        <div class="course-card">
            <div class="course-title">{lang_data['title']}</div>
            <p>{lang_data['description']}</p>
            <a class="btn" href="/course/{lang_key}">Открыть курс ➡</a>
        </div>
        """
    
    html_content = f"""
    <html>
    <head>{MOBILE_STYLES}</head>
    <body>
        <div class="container">
            <h1>Выбор направления 💻</h1>
            <p style="text-align:center; font-size:18px; margin-bottom:30px;">Какое оружие выберешь для покорения IT?</p>
            {cards_html}
            <a href="/" class="back-btn">⬅ На главную</a>
        </div>
    </body>
    </html>
    """
    return render_template_string(html_content)

# 3. СТРАНИЦА КУРСА
@app.route('/course/<lang_name>', methods=['GET', 'POST'])
def view_course(lang_name):
    course = COURSES_DATA.get(lang_name)
    if not course:
        return "Курс не найден", 404

    result_message = ""
    if request.method == 'POST':
        user_answer = request.form.get('answer')
        if user_answer == course['quiz']['correct']:
            result_message = f"<p style='color:#00ff66; font-weight:bold; font-size:20px; text-align:center; margin-top:15px;'>✅ Правильно! {course['quiz']['explain']}</p>"
        elif user_answer:
            result_message = "<p style='color:#ff3366; font-weight:bold; font-size:20px; text-align:center; margin-top:15px;'>❌ Неверно. Попробуй еще раз!</p>"

    lessons_html = ""
    for lesson in course['lessons']:
        lessons_html += f"""
        <div style="background:#1e1e1e; padding:20px; border-radius:12px; margin-bottom:15px; border:1px solid #333;">
            <h3 style="color:#00ffcc; margin-top:0; font-size:22px;">{lesson['title']}</h3>
            <p style="font-size:18px; color:#f0f0f0; margin-bottom:0;">{lesson['content']}</p>
        </div>
        """

    options_html = ""
    for opt in course['quiz']['options']:
        options_html += f"""
        <label class="option">
            <input type="radio" name="answer" value="{opt}"> <span>{opt}</span>
        </label>
        """

    html_content = f"""
    <html>
    <head>{MOBILE_STYLES}</head>
    <body>
        <div class="container">
            <h1>{course['title']}</h1>
            
            <h2 style="color:#fff; margin-top:25px;">📺 Видео-урок:</h2>
            <div style="margin-bottom:25px;">
                <video controls poster="https://images.unsplash.com/photo-1515879218367-8466d910aaa4?w=500">
                    <source src="{course['video_url']}" type="video/mp4">
                    Ваш телефон не поддерживает видео.
                </video>
            </div>
            
            <h2 style="color:#fff; margin-top:25px;">📚 Теория курса:</h2>
            {lessons_html}
            
            <h2 style="color:#fff; margin-top:25px;">🧠 Проверка знаний:</h2>
            <div style="background:#1e1e1e; padding:20px; border-radius:12px; border:1px solid #333; margin-bottom:20px;">
                <h3 style="margin-top:0; font-size:20px; color:#fff;">{course['quiz']['question']}</h3>
                <form method="POST">
                    {options_html}
                    <button type="submit" class="submit-btn">Проверить ответ</button>
                </form>
                {result_message}
            </div>
            
            <a href="/coding_hub" class="back-btn">⬅ Назад к выбору языков</a>
        </div>
    </body>
    </html>
    """
    return render_template_string(html_content)

# База данных вопросов в стиле Duolingo для английского
ENGLISH_QUIZ = [
    {
        "question": "Что означает глагол 'to Build' в контексте разработки программ?",
        "options": ["Построить дом", "Скомпилировать (собрать) код", "Найти ошибку в приложении"],
        "correct": "Скомпилировать (собрать) код"
    },
    {
        "question": "Как переводится слово 'Bug' (баг) на самом деле?",
        "options": ["Фича", "Жук / Ошибка в коде", "Новая кнопка"],
        "correct": "Жук / Ошибка в коде"
    },
    {
        "question": "Что программисты делают, когда говорят 'Deploy' (задеплоить) сайт?",
        "options": ["Удаляют его", "Пишут к нему инструкции", "Выкладывают на сервер для всех"],
        "correct": "Выкладывают на сервер для всех"
    }
]

# 4. СТРАНИЦА АНГЛИЙСКОГО В СТИЛЕ DUOLINGO
@app.route('/english', methods=['GET', 'POST'])
def view_english():
    # Получаем номер текущего вопроса из ссылки (по умолчанию 0 — первый вопрос)
    step = request.args.get('step', default=0, type=int)
    
    # Если вопросы закончились, показываем экран победы
    if step >= len(ENGLISH_QUIZ):
        return render_template_string(f"""
        <html>
        <head>{MOBILE_STYLES}</head>
        <body>
            <div class="container" style="text-align:center; margin-top:50px;">
                <h1 style="color:#00ff66;">🎉 Курс пройден!</h1>
                <p style="font-size:22px;">Ты укрепил свои позиции и стал на шаг ближе к зарубежному IT.</p>
                <a href="/" class="btn" style="background:#ff3366; color:white;">На главную</a>
            </div>
        </body>
        </html>
        """)

    current_q = ENGLISH_QUIZ[step]
    result_message = ""
    next_button = ""
    show_form = True

    if request.method == 'POST':
        user_answer = request.form.get('answer')
        if user_answer == current_q['correct']:
            result_message = "<p style='color:#00ff66; font-weight:bold; font-size:22px; text-align:center;'>✅ Великолепно! Идем дальше.</p>"
            # Появляется кнопка перехода к следующему вопросу
            next_button = f'<a href="/english?step={step + 1}" class="btn" style="background:#00ff66; color:#121212;">Продолжить ➡</a>'
            show_form = False
        elif user_answer:
            # Секретный режим: не говорим правильный ответ при ошибке
            result_message = "<p style='color:#ff3366; font-weight:bold; font-size:20px; text-align:center;'>❌ Неверно. Подумай еще раз и выбери другой вариант!</p>"

    # Генерируем варианты ответов
    options_html = ""
    for opt in current_q['options']:
        options_html += f"""
        <label class="option">
            <input type="radio" name="answer" value="{opt}"> <span>{opt}</span>
        </label>
        """

    # Форма ввода (скрывается, если ответ уже правильный, чтобы нажать "Дальше")
    form_html = f"""
    <form method="POST">
        {options_html}
        <button type="submit" class="submit-btn english-submit">Проверить ответ</button>
    </form>
    """ if show_form else ""

    html_content = f"""
    <html>
    <head>{MOBILE_STYLES}</head>
    <body>
        <div class="container">
            <h1 style="color: #ff3366;">🇬🇧 IT-Duolingo (Вопрос {step + 1} из {len(ENGLISH_QUIZ)})</h1>
            
            <div style="width:100%; background:#333; height:12px; border-radius:6px; margin-bottom:30px; overflow:hidden;">
                <div style="width:{(step) / len(ENGLISH_QUIZ) * 100}%; background:#ff3366; height:100%; transition: 0.3s;"></div>
            </div>

            <div style="background:#1e1e1e; padding:20px; border-radius:12px; border:1px solid #333;">
                <h3 style="margin-top:0; font-size:22px; color:#fff;">{current_q['question']}</h3>
                {form_html}
                {result_message}
                {next_button}
            </div>
            
            <a href="/" class="back-btn">⬅ Выйти в меню</a>
        </div>
    </body>
    </html>
    """
    return render_template_string(html_content)


if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
    
