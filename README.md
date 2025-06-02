import pandas as pd
import ipywidgets as widgets
from IPython.display import display, clear_output, FileLink
from dataclasses import dataclass, field, asdict
from typing import List
import os
from uuid import uuid4
import io

# النصوص حسب اللغة
TEXTS = {
    'ar': {
        'name': 'الاسم',
        'position': 'المركز',
        'cards': 'البطاقات',
        'interceptions': 'التدخلات الناجحة',
        'defending': 'الدفاع عن المرمى',
        'blocks': 'اعتراض المهاجمين',
        'goal_prevention': 'حراسة المرمى',
        'negative_goals': 'الأهداف السلبية',
        'passes': 'التمريرات',
        'assists': 'صناعة الأهداف',
        'shots': 'التسديدات',
        'goals': 'الأهداف',
        'matches': 'عدد المباريات',
        'add_player': 'إضافة لاعب',
        'show_table': 'عرض اللاعبين',
        'clear_all': 'مسح الكل',
        'validation_error': '⚠ الرجاء إدخال الاسم والمركز',
        'no_players': 'لا يوجد لاعبين',
        'rating': 'التقييم',
        'position_choices': ['حارس', 'مدافع', 'وسط', 'مهاجم'],
        'card_choices': ['لا بطاقة', 'بطاقة صفراء','بطاقتين صفراء', 'بطاقة حمراء'],
        'language': 'اللغة',
        'export': 'تصدير',
        'import': 'استيراد',
        'export_success': 'تم تصدير البيانات إلى الملف:',
        'import_success': 'تم استيراد {} لاعب/لاعبة بنجاح',
        'file_select_error': '⚠ يرجى اختيار ملف صحيح'
    },
    'en': {
        'name': 'Name',
        'position': 'Position',
        'cards': 'Cards',
        'interceptions': 'Interceptions',
        'defending': 'Defending',
        'blocks': 'Blocks',
        'goal_prevention': 'Goal Prevention',
        'negative_goals': 'Negative Goals',
        'passes': 'Passes',
        'assists': 'Assists',
        'shots': 'Shots',
        'goals': 'Goals',
        'matches': 'Matches',
        'add_player': 'Add Player',
        'show_table': 'Show Players',
        'clear_all': 'Clear All',
        'validation_error': '⚠ Please enter Name and Position',
        'no_players': 'No players',
        'rating': 'Rating',
        'position_choices': ['Goalkeeper', 'Defender', 'Midfielder', 'Forward'],
        'card_choices': ['No Card', 'Yellow Card','Two Yellow Cards', 'Red Card'],
        'language': 'Language',
        'export': 'Export',
        'import': 'Import',
        'export_success': 'Data exported to file:',
        'import_success': '{} players imported successfully',
        'file_select_error': '⚠ Please select a valid file'
    }
}

current_lang = 'en'

def get_text(key):
    return TEXTS[current_lang][key]

@dataclass
class Player:
    name: str
    position: str
    card: str
    interceptions: int
    defending: int
    blocks: int
    goal_prevention: int
    negative_goals: int
    passes: int
    assists: int
    shots: int
    goals: int
    matches: int
    rating: float = field(init=False, default=0.0)

    def calc_rating(self):
        w = {
            'Goalkeeper': {'goals':0, 'assists':0.1, 'passes':0.3, 'interceptions':0.7, 'shots':0, 'defending':1.5, 'blocks':0.7, 'goal_prevention':2.0, 'negative_goals':-2.5, 'cards':-1},
            'Defender': {'goals':0.3, 'assists':0.5, 'passes':0.6, 'interceptions':1.2, 'shots':0.1, 'defending':1.0, 'blocks':1.5, 'goal_prevention':0.7, 'negative_goals':-1.5, 'cards':-1},
            'Midfielder': {'goals':1.0, 'assists':1.0, 'passes':1.2, 'interceptions':0.8, 'shots':0.6, 'defending':0.5, 'blocks':0.5, 'goal_prevention':0.3, 'negative_goals':-1.0, 'cards':-1.5},
            'Forward': {'goals':2.0, 'assists':1.5, 'passes':0.5, 'interceptions':0.2, 'shots':1.5, 'defending':0.1, 'blocks':0.1, 'goal_prevention':0.1, 'negative_goals':-0.5, 'cards':-2},
            'حارس': {'goals':0, 'assists':0.1, 'passes':0.3, 'interceptions':0.7, 'shots':0, 'defending':1.5, 'blocks':0.7, 'goal_prevention':2.0, 'negative_goals':-2.5, 'cards':-1},
            'مدافع': {'goals':0.3, 'assists':0.5, 'passes':0.6, 'interceptions':1.2, 'shots':0.1, 'defending':1.0, 'blocks':1.5, 'goal_prevention':0.7, 'negative_goals':-1.5, 'cards':-1},
            'وسط': {'goals':1.0, 'assists':1.0, 'passes':1.2, 'interceptions':0.8, 'shots':0.6, 'defending':0.5, 'blocks':0.5, 'goal_prevention':0.3, 'negative_goals':-1.0, 'cards':-1.5},
            'مهاجم': {'goals':2.0, 'assists':1.5, 'passes':0.5, 'interceptions':0.2, 'shots':1.5, 'defending':0.1, 'blocks':0.1, 'goal_prevention':0.1, 'negative_goals':-0.5, 'cards':-2},
        }
        weights = w[self.position]
        card_penalty = 0
        if self.card in ['بطاقة صفراء', 'Yellow Card']:
         card_penalty = weights['cards'] * 1
        elif self.card in ['بطاقتين صفراء', 'Two Yellow Cards']:
         card_penalty = weights['cards'] * 2
        elif self.card in ['بطاقة حمراء', 'Red Card']:
         card_penalty = weights['cards'] * 3


        raw = (
            self.goals * weights['goals'] +
            self.assists * weights['assists'] +
            self.passes * weights['passes'] +
            self.interceptions * weights['interceptions'] +
            self.shots * weights['shots'] +
            self.defending * weights['defending'] +
            self.blocks * weights['blocks'] +
            self.goal_prevention * weights['goal_prevention'] +
            self.negative_goals * weights['negative_goals'] +
            card_penalty
        )
        self.rating = max(0, round(raw / max(1, self.matches), 2))

players: List[Player] = []

language_dropdown = widgets.Dropdown(
    options=[('English', 'en'), ('العربية', 'ar')],
    value=current_lang,
    description='🌐'
)

# تعريف عناصر الإدخال كما كانت (اختصرنا هنا لمثال فقط)

name_text = widgets.Text(description=get_text('name'))
position_dropdown = widgets.Dropdown(options=get_text('position_choices'), description=get_text('position'))
cards_dropdown = widgets.Dropdown(options=get_text('card_choices'), description=get_text('cards'))
interceptions_int = widgets.BoundedIntText(min=0, max=100, description=get_text('interceptions'))
defending_int = widgets.BoundedIntText(min=0, max=100, description=get_text('defending'))
blocks_int = widgets.BoundedIntText(min=0, max=100, description=get_text('blocks'))
goal_prevention_int = widgets.BoundedIntText(min=0, max=100, description=get_text('goal_prevention'))
negative_goals_int = widgets.BoundedIntText(min=0, max=100, description=get_text('negative_goals'))
passes_int = widgets.BoundedIntText(min=0, max=100, description=get_text('passes'))
assists_int = widgets.BoundedIntText(min=0, max=100, description=get_text('assists'))
shots_int = widgets.BoundedIntText(min=0, max=100, description=get_text('shots'))
goals_int = widgets.BoundedIntText(min=0, max=100, description=get_text('goals'))
matches_int = widgets.BoundedIntText(min=1, max=100, description=get_text('matches'))

add_button = widgets.Button(description=get_text('add_player'), button_style='success')
show_button = widgets.Button(description=get_text('show_table'), button_style='info')
clear_button = widgets.Button(description=get_text('clear_all'), button_style='warning')

export_button = widgets.Button(description=get_text('export'), button_style='primary')
import_button = widgets.FileUpload(accept='.xlsx', multiple=False)

output = widgets.Output()

def refresh_labels():
    # لتحديث النصوص عند تغيير اللغة
    name_text.description = get_text('name')
    position_dropdown.description = get_text('position')
    position_dropdown.options = get_text('position_choices')
    cards_dropdown.description = get_text('cards')
    cards_dropdown.options = get_text('card_choices')
    interceptions_int.description = get_text('interceptions')
    defending_int.description = get_text('defending')
    blocks_int.description = get_text('blocks')
    goal_prevention_int.description = get_text('goal_prevention')
    negative_goals_int.description = get_text('negative_goals')
    passes_int.description = get_text('passes')
    assists_int.description = get_text('assists')
    shots_int.description = get_text('shots')
    goals_int.description = get_text('goals')
    matches_int.description = get_text('matches')
    add_button.description = get_text('add_player')
    show_button.description = get_text('show_table')
    clear_button.description = get_text('clear_all')
    export_button.description = get_text('export')

def add_player_clicked(_):
    with output:
        clear_output()
        if not name_text.value.strip() or not position_dropdown.value:
            print(get_text('validation_error'))
            return
        try:
            p = Player(
                name=name_text.value.strip(),
                position=position_dropdown.value,
                card=cards_dropdown.value,
                interceptions=interceptions_int.value,
                defending=defending_int.value,
                blocks=blocks_int.value,
                goal_prevention=goal_prevention_int.value,
                negative_goals=negative_goals_int.value,
                passes=passes_int.value,
                assists=assists_int.value,
                shots=shots_int.value,
                goals=goals_int.value,
                matches=matches_int.value
            )
            p.calc_rating()
            players.append(p)
            print(f"✅ {p.name} added.")
            clear_inputs()
        except Exception as e:
            print(f"Error: {e}")

def clear_inputs():
    name_text.value = ''
    position_dropdown.value = get_text('position_choices')[0]
    cards_dropdown.value = get_text('card_choices')[0]
    interceptions_int.value = 0
    defending_int.value = 0
    blocks_int.value = 0
    goal_prevention_int.value = 0
    negative_goals_int.value = 0
    passes_int.value = 0
    assists_int.value = 0
    shots_int.value = 0
    goals_int.value = 0
    matches_int.value = 1

def rating_description(rating: float) -> str:
    # تحديد التقييم الوصفي حسب القيمة الرقمية، مع دعم اللغة
    if current_lang == 'ar':
        if rating < 3:
            return "ضعيف"
        elif rating < 6:
            return "متوسط"
        elif rating < 8:
            return "جيد"
        else:
            return "جيد جداً"
    else:  # الإنجليزية
        if rating < 3:
            return "Poor"
        elif rating < 6:
            return "Average"
        elif rating < 8:
            return "Good"
        else:
            return "Very Good"

def show_players_table():
    with output:
        clear_output()
        if not players:
            print(get_text('no_players'))
            return
        df = pd.DataFrame([{
            get_text('name'): p.name,
            get_text('position'): p.position,
            get_text('cards'): p.card,
            get_text('interceptions'): p.interceptions,
            get_text('defending'): p.defending,
            get_text('blocks'): p.blocks,
            get_text('goal_prevention'): p.goal_prevention,
            get_text('negative_goals'): p.negative_goals,
            get_text('passes'): p.passes,
            get_text('assists'): p.assists,
            get_text('shots'): p.shots,
            get_text('goals'): p.goals,
            get_text('matches'): p.matches,
            get_text('rating'): p.rating,
            get_text('rating') + ' ' + ('وصف' if current_lang=='ar' else 'Description'): rating_description(p.rating)
        } for p in players])
        display(df)


def clear_all_clicked(_):
    players.clear()
    with output:
        clear_output()
        print(get_text('no_players'))

def export_players_clicked(_):
    with output:
        clear_output()
        if not players:
            print(get_text('no_players'))
            return
        # تحويل بيانات اللاعبين إلى DataFrame مع الحقول المترجمة للعربية أو الانجليزية
        df = pd.DataFrame([{
            'Name': p.name,
            'Position': p.position,
            'Cards': p.card,
            'Interceptions': p.interceptions,
            'Defending': p.defending,
            'Blocks': p.blocks,
            'Goal Prevention': p.goal_prevention,
            'Negative Goals': p.negative_goals,
            'Passes': p.passes,
            'Assists': p.assists,
            'Shots': p.shots,
            'Goals': p.goals,
            'Matches': p.matches,
            'Rating': p.rating
        } for p in players])
        filename = f"players_{uuid4().hex}.xlsx"
        df.to_excel(filename, index=False)
        print(f"📦 {get_text('export_success')} {filename}")
        display(FileLink(filename))

def import_players_changed(change):
    with output:
        clear_output()
        try:
            if not import_button.value:
                print(get_text('file_select_error'))
                return
            # نحصل على الملف (الملف الأول فقط)
            uploaded_filename = next(iter(import_button.value))
            content = import_button.value[uploaded_filename]['content']
            # نقرأ الملف إكسل من الذاكرة
            excel_data = pd.read_excel(io.BytesIO(content))
            players.clear()
            for _, row in excel_data.iterrows():
                try:
                    p = Player(
                        name=str(row['Name']),
                        position=str(row['Position']),
                        card=str(row['Cards']),
                        interceptions=int(row['Interceptions']),
                        defending=int(row['Defending']),
                        blocks=int(row['Blocks']),
                        goal_prevention=int(row['Goal Prevention']),
                        negative_goals=int(row['Negative Goals']),
                        passes=int(row['Passes']),
                        assists=int(row['Assists']),
                        shots=int(row['Shots']),
                        goals=int(row['Goals']),
                        matches=int(row['Matches'])
                    )
                    p.calc_rating()
                    players.append(p)
                except Exception as e:
                    # إذا كان هناك خطأ في صف معين نستمر بالصفوف الأخرى
                    pass
            print(get_text('import_success').format(len(players)))
            show_players_table()
        except Exception as e:
            print(f"Error: {e}")

def on_language_change(change):
    global current_lang
    current_lang = change['new']
    refresh_labels()

language_dropdown.observe(on_language_change, names='value')

add_button.on_click(add_player_clicked)
show_button.on_click(lambda _: show_players_table())
clear_button.on_click(clear_all_clicked)
export_button.on_click(export_players_clicked)
import_button.observe(import_players_changed, names='value')

# ضبط القيم الابتدائية
clear_inputs()
refresh_labels()

# ترتيب عرض العناصر في الواجهة مع أزرار التصدير والاستيراد
buttons_box = widgets.HBox([
    add_button, show_button, clear_button,
    export_button, import_button,
    language_dropdown
], layout=widgets.Layout(margin='10px 0 0 0'))

inputs_box = widgets.VBox([
    name_text, position_dropdown, cards_dropdown,
    interceptions_int, defending_int, blocks_int,
    goal_prevention_int, negative_goals_int,
    passes_int, assists_int, shots_int,
    goals_int, matches_int
])

display(inputs_box, buttons_box, output)
