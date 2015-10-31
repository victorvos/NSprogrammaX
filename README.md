# NSprogrammaX
Vertrektijden van de Treinen op een ingevoerd station.
Het programma maakt gebruik van een GUI.
API: NS
Programmeur: Victor Vos
Taal: Python
Group: 5 
Class: V1P
Education: Hogeschool Utrecht
Studie: Informatica

CODE START
-----------------------------------------------

#!/usr/bin/python
# -*- coding: utf-8 -*-

import requests
import logging
import xmltodict
from datetime import datetime, timedelta
from Tkinter import *
import tkMessageBox

XML_FILENAME = __file__.rsplit('/', 1)[1].replace('py', '.xml')
NS_API_AUTH = ('me@victorvos.com', 'wkuaIzG71BHfKGGnigf9_tzJxr_3KPRVAG0AmpSCC5YZAsaTESc1QA')


def schrijf_xml(response, filename):

    bestand = open(filename, 'w')
    bestand.write(str(response))
    bestand.close()


def verwerk_xml(filenamem):

    bestand = open(filenamem, 'r')
    xml_string = bestand.read()
    return xmltodict.parse(xml_string)


def callNSAPI(req, auth_details):

    # noinspection PyBroadException
    try:
        response = requests.get(req, auth=auth_details)
        return response
    except BaseException:
        return logging.exception('')


def get_station_items(station):

    ns_api_stations = callNSAPI(b'http://webservices.ns.nl/ns-api-avt?station=' + station, NS_API_AUTH)
    xml_data = ns_api_stations.content.decode("utf-8")

    schrijf_xml(xml_data, XML_FILENAME)
    stations_dict = verwerk_xml(XML_FILENAME)

    if 'error' in stations_dict:
        raise UserWarning(stations_dict['error']['message'])
    else:
        return stations_dict['ActueleVertrekTijden']['VertrekkendeTrein']


def get_station_info(invoer_station):

    TF = ""
    nu_dt = datetime.now()

    for item in get_station_items(invoer_station):
        vertrektijd = datetime.strptime(item['VertrekTijd'], "%Y-%m-%dT%H:%M:%S+0100")
        if nu_dt < vertrektijd < nu_dt + timedelta(minutes=10):
            TF += "Vertrektijd: %s \n" % vertrektijd
            if item['VertrekSpoor']['@wijziging'] == u'true':
                TF += "Spoorwijziging ! Het nieuwe spoor is %s \n" % item['VertrekSpoor']['#text']
            else:
                TF += "Spoor: %s \n" % item['VertrekSpoor']['#text']
            TF += "Via: %s \n" % item.get('RouteTekst', '')
            TF += "Eindbestemming: %s \n\n" % item['EindBestemming']

    return TF


def clicked(station):
    if len(station) == 0:
        msg = "Station is niet ingevuld"
        tkMessageBox.showinfo(title="Reisinformatie onbeschikbaar", message=msg)
    else:
        if station:
            try:
                msg = get_station_info(station)
            except UserWarning as msg:
                pass
        else:
            msg = ''
        tkMessageBox.showinfo(title="Actuele reisinformatie", message=msg)


def window_init():

    window = Tk()
    window.configure(background="YELLOW")
    window.geometry("340x300")

    label = Label(window, text="Welkom bij de NS", background="YELLOW", foreground="BLUE")
    label.grid(row=0, column=0)

    label1 = Label(window, text="Voer hieronder uw huidige station in.", background="YELLOW", foreground="BLUE")
    label1.grid(row=4, column=0)

    label2 = Label(window, text="Met behulp van deze window kunt U", background="YELLOW", foreground="BLUE")
    label2.grid(row=1, column=0)

    label3 = Label(window, text="actuele reistijden opvragen van uw huidige station", background="YELLOW", foreground="BLUE")
    label3.grid(row=2, column=0)

    invoerStation = Entry(window, width=20)
    invoerStation.grid(row=5, columnspan=1)

    button1 = Button(window, padx=1, pady=3, text="Bekijk de actuele reisinformatie", background="YELLOW", foreground="BLUE",
                     command=(lambda: clicked(invoerStation.get())))
    button1.grid(row=6, sticky=N+E+S+W)
    

    #plaatje = PhotoImage(file="C:\Users\Anthony\PycharmProjects\untitled6\NS.gif")
    #NSLabel = Label(window, image=plaatje)
    #NSLabel.grid(row=7)

    return window

window_init().mainloop()
