# -- coding: utf-8 --

import pandas as pd
import ipywidgets as widgets
from IPython.display import display, clear_output
from dataclasses import dataclass, field
from typing import List, Literal
import matplotlib.pyplot as plt

try:
    import ipyaggrid as ipg
    TABLE_LIB = "ipyaggrid"
except ImportError:
    try:
        import qgrid
        TABLE_LIB = "qgrid"
    except ImportError:
        TABLE_LIB = "pandas"

LANG = 'ar'

TEXTS = {
    'ar': {
        'name': 'الاسم',
        'position': 'المركز',
        'cards': 'البطاقة',
        'interceptions': 'التدخلات',
        'defending': 'الدفاع',
        'blocks': 'اعتراض المهاجمين',
        'goal_prevention': 'منع الأهداف السلبية',
        'negative_goals': 'الأهداف السلبية',
        'passes': 'التمريرات',
        'assists': 'صناعة الهدف',
        'shots': 'التسديدات',
        'goals': 'الأهداف',
        'matches': 'عدد المباريات',
        'add_player': 'إضافة لاعب',
        'show_table': 'عرض اللاعبين',
        'clear_all': 'مسح جميع اللاعبين',
        'import_file': 'استيراد ملف',
        'export_file': 'تصدير ملف',
        'avg_rating': 'متوسط التقييم',
        'edit': 'تعديل',
        'delete': 'حذف',
        'validation_error': '⚠ يرجى إدخال الاسم والمركز.',
        'player_added': '✅ تم إضافة اللاعب: ',
        'all_cleared': 'تم مسح جميع اللاعبين.',
        'load_success': '✅ تم تحميل البيانات من الملف.',
        'file_not_found': 'ملف غير موجود.',
        'no_players': 'لا يوجد لاعبين لعرضهم.',
        'rating': 'التقييم النهائي',
        'rating_desc': ['سيء جدًا', 'سيء', 'متوسط', 'جيد', 'جيد جدًا', 'ممتاز'],
        'position_choices': ['حارس', 'مدافع', 'وسط', 'مهاجم'],
        'card_choices': ['لا بطاقة', 'بطاقة صفراء', 'بطاقة حمراء'],
        'view_chart': 'رسم بياني',
    },
    'en': {
        'name': 'Name',
        'position': 'Position',
        'cards': 'Cards',
        'interceptions': 'Interceptions',
        'defending': 'Defending',
        'blocks': 'Attacker Blocks',
        'goal_prevention': 'Goal Prevention',
        'negative_goals': 'Negative Goals',
        'passes': 'Passes',
        'assists': 'Assists',
        'shots': 'Shots',
        'goals': 'Goals',
        'matches': 'Matches',
        'add_player': 'Add Player',
        'show_table': 'Show Players',
        'clear_all': 'Clear All Players',
        'import_file': 'Import File',
        'export_file': 'Export File',
        'avg_rating': 'Average Rating',
        'edit': 'Edit',
        'delete': 'Delete',
        'validation_error': '⚠ Please enter Name and Position.',
        'player_added': '✅ Player added: ',
        'all_cleared': 'All players cleared.',
        'load_success': '✅ Data loaded from file.',
        'file_not_found': 'File not found.',
        'no_players': 'No players to show.',
        'rating': 'Final Rating',
        'rating_desc': ['Very Poor', 'Poor', 'Average', 'Good', 'Very Good', 'Excellent'],
        'position_choices': ['Goalkeeper', 'Defender', 'Midfielder', 'Forward'],
        'card_choices': ['No Card', 'Yellow Card', 'Red Card'],
        'view_chart': 'View Chart',
    }
}

T = TEXTS[LANG]

@dataclass
class Player:
    name: str
    position: Literal['حارس', 'مدافع', 'وسط', 'مهاجم', 'Goalkeeper', 'Defender', 'Midfielder', 'Forward']
    card: Literal['لا بطاقة', 'بطاقة صفراء', 'بطاقة حمراء', 'No Card', 'Yellow Card', 'Red Card']
    interceptions: int
    passes: int
    assists: int
    shots: int
    goals: int
    defending: int
    blocks: int
    goal_prevention: int
    negative_goals: int
    matches: int
    rating: float = field(init=False, default=0.0)

    def calculate_rating(self):
        weights = {
            'حارس': {'goals': 0, 'assists': 0.2, 'passes': 0.5, 'interceptions': 0.7, 'shots': 0,
                     'defending': 2.0, 'blocks': 0.5, 'goal_prevention': 3.0, 'negative_goals': -2.0, 'cards': -1},
            'مدافع': {'goals': 0.5, 'assists': 0.4, 'passes': 0.6, 'interceptions': 2.0, 'shots': 0.3,
                     'defending': 1.8, 'blocks': 2.5, 'goal_prevention': 1.0, 'negative_goals': -1.5, 'cards': -1.5},
            'وسط': {'goals': 1.2, 'assists': 1.5, 'passes': 1.2, 'interceptions': 1.0, 'shots': 0.8,
                    'defending': 1.0, 'blocks': 1.0, 'goal_prevention': 0.5, 'negative_goals': -1.0, 'cards': -2},
            'مهاجم': {'goals': 2.0, 'assists': 1.0, 'passes': 0.6, 'interceptions': 0.5, 'shots': 1.7,
                    'defending': 0.2, 'blocks': 0.3, 'goal_prevention': 0.1, 'negative_goals': -3.0, 'cards': -3},
            'Goalkeeper': {'goals': 0, 'assists': 0.2, 'passes': 0.5, 'interceptions': 0.7, 'shots': 0,
                           'defending': 2.0, 'blocks': 0.5, 'goal_prevention': 3.0, 'negative_goals': -2.0, 'cards': -1},
            'Defender': {'goals': 0.5, 'assists': 0.4, 'passes': 0.6, 'interceptions': 2.0, 'shots': 0.3,
                         'defending': 1.8, 'blocks': 2.5, 'goal_prevention': 1.0, 'negative_goals': -1.5, 'cards': -1.5},
            'Midfielder': {'goals': 1.2, 'assists': 1.5, 'passes': 1.2, 'interceptions': 1.0, 'shots': 0.8,
                           'defending': 1.0, 'blocks': 1.0, 'goal_prevention': 0.5, 'negative_goals': -1.0, 'cards': -2},
            'Forward': {'goals': 2.0, 'assists': 1.0, 'passes': 0.6, 'interceptions': 0.5, 'shots': 1.7,
                        'defending': 0.2, 'blocks': 0.3, 'goal_prevention': 0.1, 'negative_goals': -3.0, 'cards': -3},
        }
        w = weights[self.position]

        card_penalty = 0
        if self.card in ['بطاقة صفراء', 'Yellow Card']:
            card_penalty = w['cards'] * 1
        elif self.card in ['بطاقة حمراء', 'Red Card']:
            card_penalty = w['cards'] * 3

        self.rating = max(0, round(
            self.goals * w['goals'] +
            self.assists * w['assists'] +
            self.passes * w['passes'] +
            self.interceptions * w['interceptions'] +
            self.shots * w['shots'] +
            self.defending * w['defending'] +
            self.blocks * w['blocks'] +
            self.goal_prevention * w['goal_prevention'] +
            self.negative_goals * w['negative_goals'] +
            card_penalty, 2
        ))

def rating_description(score: float) -> str:
    # مقياس توضيحي حسب التقييم من 0 إلى 20 (مثلاً)
    if score < 4:
        return T['rating_desc'][0]  # سيء جدًا
    elif score < 8:
        return T['rating_desc'][1]  # سيء
    elif score < 12:
        return T['rating_desc'][2]  # متوسط
    elif score < 16:
        return T['rating_desc'][3]  # جيد
    elif score < 18:
        return T['rating_desc'][4]  # جيد جدًا
    else:
        return T['rating_desc'][5]  # ممتاز

# القائمة الرئيسية والواجهة

players: List[Player] = []

inputs = {}

def build_inputs():
    inputs['name'] = widgets.Text(description=T['name'], layout=widgets.Layout(width='300px'))
    inputs['position'] = widgets.Dropdown(options=T['position_choices'], description=T['position'], layout=widgets.Layout(width='300px'))
    inputs['card'] = widgets.Dropdown(options=T['card_choices'], description=T['cards'], layout=widgets.Layout(width='300px'))
    inputs['interceptions'] = widgets.BoundedIntText(value=0, min=0, max=100, description=T['interceptions'], layout=widgets.Layout(width='300px'))
    inputs['defending'] = widgets.BoundedIntText(value=0, min=0, max=10, description=T['defending'], layout=widgets.Layout(width='300px'))
    inputs['blocks'] = widgets.BoundedIntText(value=0, min=0, max=10, description=T['blocks'], layout=widgets.Layout(width='300px'))
    inputs['goal_prevention'] = widgets.BoundedIntText(value=0, min=0, max=10, description=T['goal_prevention'], layout=widgets.Layout(width='300px'))
    inputs['negative_goals'] = widgets.BoundedIntText(value=0, min=0, max=10, description=T['negative_goals'], layout=widgets.Layout(width='300px'))
    inputs['passes'] = widgets.BoundedIntText(value=0, min=0, max=1000, description=T['passes'], layout=widgets.Layout(width='300px'))
    inputs['assists'] = widgets.BoundedIntText(value=0, min=0, max=1000, description=T['assists'], layout=widgets.Layout(width='300px'))
    inputs['shots'] = widgets.BoundedIntText(value=0, min=0, max=1000, description=T['shots'], layout=widgets.Layout(width='300px'))
    inputs['goals'] = widgets.BoundedIntText(value=0, min=0, max=1000, description=T['goals'], layout=widgets.Layout(width='300px'))
    inputs['matches'] = widgets.BoundedIntText(value=1, min=1, max=1000, description=T['matches'], layout=widgets.Layout(width='300px'))

def add_player(_):
    clear_output(wait=True)
    if not inputs['name'].value or not inputs['position'].value:
        print(T['validation_error'])
        display(ui)
        return

    player = Player(
        name=inputs['name'].value,
        position=inputs['position'].value,
        card=inputs['card'].value,
        interceptions=inputs['interceptions'].value,
        defending=inputs['defending'].value,
        blocks=inputs['blocks'].value,
        goal_prevention=inputs['goal_prevention'].value,
        negative_goals=inputs['negative_goals'].value,
        passes=inputs['passes'].value,
        assists=inputs['assists'].value,
        shots=inputs['shots'].value,
        goals=inputs['goals'].value,
        matches=inputs['matches'].value,
    )
    player.calculate_rating()
    players.append(player)
    print(f"{T['player_added']}{player.name} - {T['rating']}: {player.rating} ({rating_description(player.rating)})")
    display(ui)

def show_players(_):
    clear_output(wait=True)
    if not players:
        print(T['no_players'])
        display(ui)
        return

    df = pd.DataFrame([{
        T['name']: p.name,
        T['position']: p.position,
        T['cards']: p.card,
        T['interceptions']: p.interceptions,
        T['defending']: p.defending,
        T['blocks']: p.blocks,
        T['goal_prevention']: p.goal_prevention,
        T['negative_goals']: p.negative_goals,
        T['passes']: p.passes,
        T['assists']: p.assists,
        T['shots']: p.shots,
        T['goals']: p.goals,
        T['matches']: p.matches,
        T['rating']: f"{p.rating} ({rating_description(p.rating)})"
    } for p in players])
    
    display(df)
    display(ui)

def clear_all(_):
    clear_output(wait=True)
    players.clear()
    print(T['all_cleared'])
    display(ui)

def export_file(_):
    if not players:
        print(T['no_players'])
        display(ui)
        return
    df = pd.DataFrame([{
        'name': p.name,
        'position': p.position,
        'card': p.card,
        'interceptions': p.interceptions,
        'defending': p.defending,
        'blocks': p.blocks,
        'goal_prevention': p.goal_prevention,
        'negative_goals': p.negative_goals,
        'passes': p.passes,
        'assists': p.assists,
        'shots': p.shots,
        'goals': p.goals,
        'matches': p.matches,
        'rating': p.rating,
    } for p in players])
    df.to_excel('players_export.xlsx', index=False)
    print('✅ تم تصدير البيانات إلى ملف players_export.xlsx')
    display(ui)

def import_file(_):
    try:
        df = pd.read_excel('players_export.xlsx')
    except FileNotFoundError:
        print(T['file_not_found'])
        display(ui)
        return
    
    players.clear()
    for _, row in df.iterrows():
        player = Player(
            name=row['name'],
            position=row['position'],
            card=row['card'],
            interceptions=int(row['interceptions']),
            defending=int(row['defending']),
            blocks=int(row['blocks']),
            goal_prevention=int(row['goal_prevention']),
            negative_goals=int(row['negative_goals']),
            passes=int(row['passes']),
            assists=int(row['assists']),
            shots=int(row['shots']),
            goals=int(row['goals']),
            matches=int(row['matches']),
        )
        player.calculate_rating()
        players.append(player)
    clear_output(wait=True)
    print(T['load_success'])
    display(ui)

def show_chart(_):
    clear_output(wait=True)
    if not players:
        print(T['no_players'])
        display(ui)
        return

    # لرسم بيانات اللاعب الأول كمثال
    p = players[-1]  # آخر لاعب مضاف
    categories = [T['goals'], T['assists'], T['passes'], T['interceptions'], T['defending'], T['blocks'], T['goal_prevention'], T['negative_goals']]
    values = [p.goals, p.assists, p.passes, p.interceptions, p.defending, p.blocks, p.goal_prevention, p.negative_goals]

    plt.figure(figsize=(7,7))
    ax = plt.subplot(111, polar=True)
    angles = [n / float(len(categories)) * 2 * 3.14159 for n in range(len(categories))]
    values += values[:1]  # إغلاق الدائرة
    angles += angles[:1]
    ax.plot(angles, values, 'o-', linewidth=2)
    ax.fill(angles, values, alpha=0.25)
    ax.set_thetagrids([a * 180/3.14159 for a in angles[:-1]], categories)
    ax.set_ylim(0, 10)
    plt.title(f"{T['view_chart']} - {p.name}")
    plt.show()
    display(ui)

# بناء الواجهة
build_inputs()

# بناء الواجهة
build_inputs()

btn_add = widgets.Button(description=T['add_player'], button_style='success')
btn_show = widgets.Button(description=T['show_table'], button_style='info')
btn_clear = widgets.Button(description=T['clear_all'], button_style='warning')
btn_import = widgets.Button(description=T['import_file'])
btn_export = widgets.Button(description=T['export_file'])
btn_chart = widgets.Button(description=T['view_chart'])

btn_add.on_click(add_player)
btn_show.on_click(show_players)
btn_clear.on_click(clear_all)
btn_import.on_click(import_file)
btn_export.on_click(export_file)
btn_chart.on_click(show_chart)

input_widgets = [
    inputs['name'], inputs['position'], inputs['card'], inputs['interceptions'], inputs['defending'],
    inputs['blocks'], inputs['goal_prevention'], inputs['negative_goals'], inputs['passes'],
    inputs['assists'], inputs['shots'], inputs['goals'], inputs['matches']
]

buttons = widgets.HBox([btn_add, btn_show, btn_clear, btn_import, btn_export, btn_chart])

ui = widgets.VBox(input_widgets + [buttons])

display(ui)

