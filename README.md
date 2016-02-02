# minion

print('executing minion.minion_magnetstage_control')

import os
import time
from PyQt5.QtCore import *
from PyQt5.QtWidgets import *
import numpy as np
import matplotlib as plt
#import matplotlib.style as pltstyle
import serial
from ctypes import *


class MinionMagnetStage(QWidget):
    def __int__(self, parent):
        super(MinionMagnetStage, self).__init__(parent)
        self.parent = parent

        self.initUI()

        # ComboBox lists
        self.units = ['mm', 'um']
        self.measurementprogrammes = ['Measure fluorescence and path', 'Ground state LAC', 'Excited state LAC']

        # variables
        self.stepsizezy = 5.0
        self.stepsizex = 3.175
        self.stepsizetheta = 1.8

    def initUI(self):

        # -------- create status labels and status windows -------------------------------------------------------------

        # display stage position
        self.labelstageposition = QLabel('current stage position')

        self.labelstatusz = QLabel('z')
        self.statusz = QTextEdit()
        self.statusz.setReadOnly(True)

        self.labelstatusy = QLabel('y')
        self.statusy = QTextEdit()
        self.statusy.setReadOnly(True)

        self.labelstatusx = QLabel('x')
        self.statusx = QTextEdit()
        self.statusx.setReadOnly(True)

        self.labelstatustheta = QLabel(r'$\Theta$')
        self.statustheta = QTextEdit()
        self.statustheta.setReadOnly(True)

        # display stopper status
        self.labelstatusstopper = QLabel('status of stoppers')

        self.labelstatusstopperz1 = QLabel('z start')
        self.statusstopperz1 = QTextEdit()
        self.statusstopperz1.setReadOnly(True)

        self.labelstatusstopperz2 = QLabel('z end')
        self.statusstopperz2 = QTextEdit()
        self.statusstopperz2.setReadOnly(True)

        self.labelstatusstoppery1 = QLabel('y start')
        self.statusstoppery1 = QTextEdit()
        self.statusstoppery1.setReadOnly(True)

        self.labelstatusstoppery2 = QLabel('y end')
        self.statusstoppery2 = QTextEdit()
        self.statusstoppery2.setReadOnly(True)

        self.labelstatusstopperx1 = QLabel('x start')
        self.statusstopperx1 = QTextEdit()
        self.statusstopperx1.setReadOnly(True)

        self.labelstatusstopperx2 = QLabel('x end')
        self.statusstopperx2 = QTextEdit()
        self.statusstopperx2.setReadOnly(True)

        self.labelstatusstoppertheta = QLabel('theta')
        self.statusstoppertheta = QTextEdit()
        self.statusstoppertheta.setReadOnly(True)

        # -------- create input labels, windows and buttons ------------------------------------------------------------

        self.labelinput = QLabel('input displacement')

        # z
        self.labelinputz = QLabel('z')
        self.inputz = QLineEdit()
        self.labelinputinz = QLabel('in')
        self.chooseunitzbox = QComboBox()
        self.chooseunitzbox.addItems(self.units)
        self.chooseunitzbox.currentIndexChanged.connect(self.chooseunitz)
        self.buttonmovestagez = QPushButton('movestage')
        self.buttonmovestagez.clicked.connect(self.movestagez)

        # y
        self.labelinputy = QLabel('y')
        self.inputy = QLineEdit()
        self.labelinputiny = QLabel('in')
        self.chooseunitybox = QComboBox()
        self.chooseunitybox.addItems(self.units)
        self.chooseunitybox.currentIndexChanged.connect(self.chooseunity)
        self.buttonmovestagey = QPushButton('movestage')
        self.buttonmovestagey.clicked.connect(self.movestagey)

        # x
        self.labelinputx = QLabel('x')
        self.inputx = QLineEdit()
        self.labelinputinx = QLabel('in')
        self.chooseunitxbox = QComboBox()
        self.chooseunitxbox.addItems(self.units)
        self.chooseunitxbox.currentIndexChanged.connect(self.chooseunitx)
        self.buttonmovestagex = QPushButton('movestage')
        self.buttonmovestagex.clicked.connect(self.movestagex)

        # theta
        self.labelinputtheta = QLabel(r'$\Theta$')
        self.inputtheta = QLineEdit()
        self.labelinputintheta = QLabel('in')
        self.buttonmovestagetheta = QPushButton('movestage')
        self.buttonmovestagetheta.clicked.connect(self.movestagetheta)

        # -------- create measurement labels, Edits, Buttons -----------------------------------------------------------

        self.labelmeasurementheader = QLabel('choose preprogrammed path or measurement')
        self.choosemeasurement = QComboBox()
        self.choosemeasurement.addItem(self.measurementprogrammes)
        self.buttonstartmeasurement = QPushButton('start process')
        self.buttonstartmeasurement.clicked.connect(self.startmeasurement)
        self.labelmeasurementstatus = QLabel('process status')
        self.measurementstatus = QProgressBar()
        self.measurementstatus.setGeometry()
        self.labelsavemeasurement = QLabel('save file under')
        self.savemeasurementfilename = QLineEdit()
        self.savemeasurementfilename.setText('filename')
        self.buttonsavemeasurement = QPushButton('save measurment')
        self.buttonsavemeasurement.clicked.connect(self.savemeasurementclicked)

        # -------- create mode buttons ---------------------------------------------------------------------------------

        self.buttoncubicmagnet = QPushButton('cubic magnet')
        self.buttoncubicmagnet.clicked.connect(self.cubicmagnet)

        self.buttontriaxialmagent = QPushButton('triaxial magnet')
        self.buttontriaxialmagent.clicked.connect(self.triaxialmagnet)

        # -------- create layout ---------------------------------------------------------------------------------------

        magnetstage_layout = QGridLayout()

        modebuttonlayout = QHBoxLayout()
        modebuttonlayout.addWidget(self.buttoncubicmagnet)
        modebuttonlayout.addWidget(self.buttontriaxialmagent)
        magnetstage_layout.addLayout(modebuttonlayout, 0, 0)

        # layout stage postion

        stagepostionlayout = QGridlayout()
        stagepostionlayout.addWidget(self.labelstageposition, 0, 0)

        stagepostionlayoutz = QHBoxLayout()
        stagepostionlayoutz.addWidget(self.labelstatusz)
        stagepostionlayoutz.addWidget(self.statusz)
        stagepostionlayout.addLayout(stagepostionlayoutz, 1, 0)

        stagepostionlayouty = QHBoxLayout()
        stagepostionlayouty.addWidget(self.labelstatusy)
        stagepostionlayouty.addWidget(self.statusy)
        stagepostionlayout.addLayout(stagepostionlayouty, 2, 0)

        stagepostionlayoutx = QHBoxLayout()
        stagepostionlayoutx.addWidget(self.labelstatusx)
        stagepostionlayoutx.addWidget(self.statusx)
        stagepostionlayout.addLayout(stagepostionlayoutx, 3, 0)

        stagepostionlayouttheta = QHBoxLayout()
        stagepostionlayouttheta.addWidget(self.labelstatustheta)
        stagepostionlayouttheta.addWidget(self.statustheta)
        stagepostionlayout.addLayout(stagepostionlayouttheta, 4, 0)

        magnetstage_layout.addLayout(stagepostionlayout, 1, 0)

        # layout stopper status

        stopperlayout = QGridLayout()
        stopperlayout.addWidget(self.labelstatusstopper, 0, 0)

        stopperlayoutz = QHBoxLayout()
        stopperlayoutz.addWidget(self.labelstatusstopperz1)
        stopperlayoutz.addWidget(self.statusstopperz1)
        stopperlayoutz.addWidget(self.labelstatusstopperz2)
        stopperlayoutz.addWidget(self.statusstopperz2)
        stopperlayout.addLayout(stagepostionlayoutz, 1, 0)

        stopperlayouty = QHBoxLayout()
        stopperlayouty.addWidget(self.labelstatusstoppery1)
        stopperlayouty.addWidget(self.statusstoppery1)
        stopperlayouty.addWidget(self.labelstatusstoppery2)
        stopperlayouty.addWidget(self.statusstoppery2)
        stopperlayout.addLayout(stagepostionlayouty, 2, 0)

        stopperlayoutx = QHBoxLayout()
        stopperlayoutx.addWidget(self.labelstatusstopperx1)
        stopperlayoutx.addWidget(self.statusstopperx1)
        stopperlayoutx.addWidget(self.labelstatusstopperx2)
        stopperlayoutx.addWidget(self.statusstopperx2)
        stopperlayout.addLayout(stagepostionlayoutx, 3, 0)

        stopperlayouttheta = QHBoxLayout()
        stopperlayouttheta.addWidget(self.labelstatusstoppertheta)
        stopperlayouttheta.addWidget(self.statusstoppertheta)
        stopperlayout.addLayout(stagepostionlayouttheta, 4, 0)

        magnetstage_layout.addLayout(stopperlayout, 1, 1)

        # layout input

        inputlayout = QGridLayout()
        inputlayout.addWidget(self.labelinput, 0, 0)

        inputzlayout = QHBoxLayout()
        inputzlayout.addWidget(self.labelinputz)
        inputzlayout.addWidget(self.inputz)
        inputzlayout.addWidget(self.labelinputinz)
        inputzlayout.addWidget(self.chooseunity)
        inputzlayout.addWidget(self.buttonmovestagez)
        inputlayout.addLayout(inputzlayout, 1, 0)

        inputylayout = QHBoxLayout()
        inputylayout.addWidget(self.labelinputy)
        inputylayout.addWidget(self.inputy)
        inputylayout.addWidget(self.labelinputiny)
        inputylayout.addWidget(self.chooseunity)
        inputylayout.addWidget(self.buttonmovestagey)
        inputlayout.addLayout(inputylayout, 2, 0)

        inputxlayout = QHBoxLayout()
        inputxlayout.addWidget(self.labelinputx)
        inputxlayout.addWidget(self.inputx)
        inputxlayout.addWidget(self.labelinputinx)
        inputxlayout.addWidget(self.chooseunitx)
        inputxlayout.addWidget(self.buttonmovestagex)
        inputlayout.addLayout(inputxlayout, 3, 0)

        inputthetalayout = QHBoxLayout()
        inputthetalayout.addWidget(self.labelinputtheta)
        inputthetalayout.addWidget(self.inputtheta)
        inputthetalayout.addWidget(self.labelinputintheta)
        inputthetalayout.addWidget(self.buttonmovestagetheta)
        inputlayout.addLayout(inputthetalayout, 4, 0)

        magnetstage_layout.addLayout(inputlayout)

        # layout measurement

        measurementlayout = QGridLayout()
        measurementlayout.addWidget(self.labelmeasurementheader, 0, 0)

        choosemeasurementlayout = QHBoxLayout()
        choosemeasurementlayout.addWidget(self.choosemeasurement)
        choosemeasurementlayout.addWidget(self.buttonstartmeasurement)
        measurementlayout.addLayout(choosemeasurementlayout, 1, 0)

        statusmeasurementlayout = QHBoxLayout()
        statusmeasurementlayout.addWidget(self.labelmeasurementstatus)
        statusmeasurementlayout.addWidget(self.measurementstatus)
        measurementlayout.addLayout(statusmeasurementlayout, 2, 0)

        savemeasurementlayout = QHBoxLayout()
        savemeasurementlayout.addWidget(self.labelsavemeasurement)
        savemeasurementlayout.addWidget(self.savemeasurement)
        savemeasurementlayout.addWidget(self.buttonsavemeasurement)
        measurementlayout.addLayout(savemeasurementlayout, 3, 0)

        magnetstage_layout.addLayout(measurementlayout)

    def savemeasurementclicked(self):
        self.filename, *rest = self.savemeasurementfilename.text().split('.')
        # np.savetxt(str(os.getcwd())+'/data/'+str(self.filename)+'.txt', self.mapdata)
        # self.mapfigure.savefig(str(os.getcwd())+'/data/'+str(self.filename)+'.pdf')
        # self.mapfigure.savefig(str(os.getcwd())+'/data/'+str(self.filename)+'.png')
        print('file saved to data folder')

    def startmeasurement(self):

    def chooseunitz(self):

        text = self.chooseunitzbox.currentText()

        if text == 'mm':
            self.unitz = 1000
        if text == 'um':
            self.unitz = 1

    def chooseunity(self):

        text = self.chooseunitybox.currentText()

        if text == 'mm':
            self.unity = 1000
        if text == 'um':
            self.unity = 1

    def chooseunitx(self):

        text = self.chooseunitxbox.currentText()

        if text == 'mm':
            self.unitx = 1000
        if text == 'um':
            self.unitx = 1

    def movestagez(self):
        self.movestagezsteps = 0.0
        inputvaluez = float(self.inputz)
        movestagezby = inputvaluez*self.unitz   #in um

        if movestagezby > self.stepsizezy:      #check minimum of input
            if movestagezby % self.stepsizezy:  #check multiple of step size
                print('Length is possible.')
                self.movestagezsteps = movestagezby / self.stepsizezy
            else:
                print('Given length not multiple of step size. Step size is 5um.')
        else:
            print('Given length is to small. Minimum 5um.')


    def movestagey(self):
        self.movestageysteps = 0.0
        inputvaluey = float(self.inputy)
        movestageyby = inputvaluey*self.unity   #in um

        if movestageyby > self.stepsizezy:      #check minimum of input
            if movestageyby % self.stepsizezy:  #check multiple of step size
                print('Length is possible.')
                self.movestageysteps = movestageyby / self.stepsizezy
            else:
                print('Given length not multiple of step size. Step size is 5um.')
        else:
            print('Given length is to small. Minimum 5um.')

    def movestagex(self):
        self.movestageysteps = 0.0
        inputvaluex = float(self.inputx)
        movestagexby = inputvaluex*self.unitx   #in um m

        if movestagexby > self.stepsizex:       #check minimum of input
            if movestagexby % self.stepsizex:   #check multiple of step size
                print('Length is possible.')
                self.movestagexsteps = movestagexby / self.stepsizex
            else:
                print('Given length not multiple of step size. Step size is 3.175um.')
        else:
            print('Given length is to small. Minimum 3.175um.')

    def movestagetheta(self):
        self.movestagethetasteps = 0.0
        inputvaluetheta = float(self.inputtheta)
        movestagethetaby = inputvaluetheta*self.unittheta    #in um

        if movestagethetaby > self.stepsizetheta:       #check minimum of input
            if movestagethetaby % self.stepsizetheta:   #check multiple of step size
                print('Angle is possible.')
                self.movestagethetasteps = movestagethetaby / self.stepsizetheta
            else:
                print('Given angle not multiple of step size. Step size is 1.8°.')
        else:
            print('Given angle is to small. Minimum 1.8°.')
