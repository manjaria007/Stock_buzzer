import PySimpleGUI as sg
import auto_py_to_exe
from yahoo_fin import stock_info as si
from datetime import datetime
from datetime import date
import smtplib
from email.message import EmailMessage

upper_bound = []
upper_upper = []
lower_bound = []
lower_lower = []
ticker = "temporary"
check = False

sg.theme('Reds')
# sg.Window(title="Window", layout=[[]], margins=(100, 50)).read()


if si.get_market_status() == 'POST':
    layoutLeft = [
        [sg.Text('Market Status: Closed', key='-STAT-')],
        [sg.Text('Data Provided by Yahoo Finance')],
        [sg.Text('Ticker Symbol', key='-Tick-', size=(10, 1)), sg.InputText(size=(21, 1), key='-Res-')],
        [sg.Button('Submit', key='Submit')],
    ]
else:
    layoutLeft = [
        [sg.Text('Market Status: Open', key='-STAT-')],
        [sg.Text('Data Provided by Yahoo Finance')],
        [sg.Text('Ticker Symbol', key='-Tick-', size=(10, 1)), sg.InputText(size=(21, 1), key='-Res-')],
        [sg.Button('Submit', key='Submit')],
    ]

layoutRight = [
    [sg.Text('Stock Ticker: ', key='-Title-')],
    [sg.Text('Latest Price: ', key='-Price-')],
    [sg.Text('Set Stock Limits'), sg.Input(size=(19, 1), key='-LIMIT-', do_not_clear=False)],
    [sg.Button('Add Limit', key='-ADD-', disabled=True), sg.Button('Delete Limit', key='-DELETE-', disabled=True),
     sg.Button('Run Program', key='-RUN-', disabled=True)],
]

layoutLower = [
    [sg.Text('Upper Limit List: ', key='-Upper-')],
    [sg.Text('Lower Limit List: ', key='-Lower-')],
]

data = [' ']
List = ['Notification Center:']

# layout = [
#     [
#         sg.Column(layoutLeft, key='-COL1-'), sg.VerticalSeparator(), sg.Column(layoutRight, visible=True, key='-COL2-'),
#         sg.VerticalSeparator(), sg.Table(data, key='-TABLE1-', headings=['Upper Limits ($)']),
#         sg.Table(data, key='-TABLE2-', headings=['Lower Limits ($)']),
#         sg.VerticalSeparator(), sg.Multiline(data, disabled=True, size=(40, 2))
#     ]
# ]

layout = [
    [
        [sg.Column(layoutLeft, key='-COL1-'), sg.VerticalSeparator(),
         sg.Column(layoutRight, visible=True, key='-COL2-')],
        [sg.HorizontalSeparator()],
        [sg.Table(data, key='-TABLE1-', size=(20, 10), headings=['Upper Limits']),
         sg.Listbox(List, key='-NOTE-', size=(40, 11)),
         sg.Table(data, key='-TABLE2-', size=(20, 10), headings=['Lower Limits'])]

        # sg.VerticalSeparator(), sg.Table(data, key='-TABLE1-', headings=['Upper Limits ($)']),
        # sg.Table(data, key='-TABLE2-', headings=['Lower Limits ($)']),
        # sg.VerticalSeparator(), sg.Multiline(data, disabled=True, size=(40, 2))
    ]
]

window = sg.Window("Stock Beeper Application - 1st Generation - 10/2021", layout)
# window = sg.Window("Stock Beeper Application", layout, size=(1000, 500))

while True:
    if check:
        current_price = si.get_live_price(ticker)
    event, values = window.read()

    if si.get_market_status() == 'POST':
        window['-STAT-'].update('Market Status: Closed')
    else:
        window['-STAT-'].update('Market Status: Open')

    if event == sg.WIN_CLOSED:
        break
    if event == "Submit":
        ticker = values['-Res-']
        print(ticker)
        stop = True

        upper_bound.clear()
        lower_bound.clear()
        upper_upper.clear()
        lower_lower.clear()
        window['-TABLE1-'].update(upper_bound)
        window['-TABLE2-'].update(lower_bound)
        window['-Title-'].update('Stock Ticker: ')
        window['-Price-'].update('Latest Price: ')
        window['-RUN-'].update(disabled=True)
        window['-DELETE-'].update(disabled=True)

        try:
            current_price = si.get_live_price(ticker)
        except:
            stop = False
            List.append("Invalid Ticker Symbol - " + ticker)
            window['-NOTE-'].update(List)
            check = False

        if stop:
            check = True
            window['-Title-'].update('Stock Ticker: ' + ticker)
            window['-Price-'].update('Latest Price: $' + str(round(current_price, 2)))
            window['-ADD-'].update(disabled=False)

            List.append("Valid Ticker Symbol - " + ticker)
            window['-NOTE-'].update(List)

    if event == '-RUN-':
        print("Running...")

        if si.get_market_status() == 'POST':
            List.append('ERROR -> Market Status: Closed')
            window['-NOTE-'].update(List)
        else:
            while len(upper_bound) > 0 or len(lower_bound) > 0:
                if len(upper_bound) > 0:
                    curr_upper = upper_bound[len(upper_bound) - 1]
                else:
                    curr_upper = 0

                if len(lower_bound) > 0:
                    curr_lower = lower_bound[len(lower_bound) - 1]
                else:
                    curr_lower = 0

                if len(lower_bound) > 0 and si.get_live_price(ticker) <= float(curr_lower):
                    today = date.today()
                    d = today.strftime("%m/%d/%y")
                    now = datetime.now()
                    current_time = now.strftime("%H:%M")
                    List.append(
                        str(d) + '-' + current_time + ' {' + ticker + " hit $" + str(curr_lower) + " lower limit}")
                    window['-NOTE-'].update(List)

                    # Send notification
                    server = smtplib.SMTP_SSL("smtp.gmail.com", 465)
                    server.login()
                    msgMin = EmailMessage()
                    msgMin['From'] = "mohitmanjaria55333@gmail.com"
                    msgMin['To'] = "6176859015@tmomail.net"
                    msgMin['Subject'] = " Minimum Alert Message for " + ticker
                    msgMin.set_content("Current Price is: " + str(curr_lower))
                    server.send_message(msgMin)
                    server.quit()

                    print("Hit Lower Bound at " + str(curr_lower))
                    lower_bound.remove(curr_lower)
                    break

                if len(upper_bound) > 0 and si.get_live_price(ticker) >= float(curr_upper):
                    today = date.today()
                    d = today.strftime("%m/%d/%y")
                    now = datetime.now()
                    current_time = now.strftime("%H:%M")
                    List.append(
                        str(d) + '-' + current_time + ' {' + ticker + " hit $" + str(curr_upper) + " upper limit}")
                    window['-NOTE-'].update(List)

                    # Send notification
                    server = smtplib.SMTP_SSL("smtp.gmail.com", 465)
                    server.login()
                    msgMax = EmailMessage()
                    msgMax['From'] = "mohitmanjaria55333@gmail.com"
                    msgMax['To'] = "6176859015@tmomail.net"
                    msgMax['Subject'] = " Maximum Alert Message for " + ticker
                    msgMax.set_content("Current Price is: " + str(curr_upper))
                    server.send_message(msgMax)
                    server.quit()

                    print("Hit Upper Bound at " + str(curr_upper))
                    upper_bound.remove(curr_upper)
                    break
            print("Ending...")

    if event == '-ADD-':
        window['-RUN-'].update(disabled=False)
        curr = values['-LIMIT-']

        checker = True
        try:
            float(curr)
        except ValueError:
            checker = False

        if checker:
            print(curr)
            if float(curr) > float(current_price):
                window['-RUN-'].update(disabled=False)
                window['-DELETE-'].update(disabled=False)
                upper_bound.append(round(float(curr), 2))
                upper_bound.sort(reverse=True)
                upper_upper = upper_bound
                upper_upper.sort()
                window['-TABLE1-'].update(upper_upper)
            elif float(current_price) > float(curr) > -1:
                window['-RUN-'].update(disabled=False)
                window['-DELETE-'].update(disabled=False)
                lower_bound.append(round(float(curr), 2))
                lower_bound.sort(reverse=False)
                lower_lower = lower_bound
                lower_lower.sort(reverse=True)
                window['-TABLE2-'].update(lower_lower)

    if event == '-DELETE-':
        curr = values['-LIMIT-']
        try:
            float(curr)
        except ValueError:
            break
        if float(curr) in upper_bound:
            upper_bound.remove(round(float(curr), 2))
            # upper_upper.remove(round(float(curr),2))
            window['-TABLE1-'].update(upper_upper)
        elif float(curr) in lower_bound:
            lower_bound.remove(round(float(curr), 2))
            # lower_lower.remove(round(float(curr),2))
            window['-TABLE2-'].update(lower_lower)

        if len(upper_bound) == 0 and len(lower_bound) == 0:
            window['-DELETE-'].update(disabled=True)
            window['-RUN-'].update(disabled=True)

window.close()
