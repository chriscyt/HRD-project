# HRD-project
# -*- coding: utf-8 -*-
"""
Created on Thu Mar 15 15:03:15 2018

@author: chris
"""

from kivy.app import App
from kivy.uix.widget import Widget
from kivy.uix.label import Label
from kivy.uix.textinput import TextInput
from kivy.graphics import Color, Rectangle
from kivy.uix.button import Button
from kivy.uix.image import AsyncImage
import numpy as np
import cv2
import time
import pickle

source=0
num=0
recent_array=np.array([[1000]*4]*5)
answer_list=0


class StartWidget(Button):
    def on_touch_down(self, touch):
        if self.collide_point(*touch.pos):
            try:
                global recent_array
                global answer_list
                h=HRD(recent_array)
                answer_list=h.answer
                
            except:
                pass
            
class ChooseWidget(Button):
    def on_touch_down(self, touch):
        if self.collide_point(*touch.pos):            
            try:
                global source
                p=pic_translate(source)
                global recent_array
                recent_array=p.final_array
            except:
                pass
class ClearWidget(Button):
    def on_touch_down(self, touch):
        if self.collide_point(*touch.pos):   
            global recent_array
            global num
            num=0
            recent_array=np.array([[1000]*4]*5)

class NextWidget(Button):
    def on_touch_down(self, touch):
        if self.collide_point(*touch.pos):
            try:
                global num
                num+=1
                global answer_list
                global recent_array
                recent_array=answer_list[num]
            except:
                pass
            

class PreciousWidget(Button):
    def on_touch_down(self, touch):
        if self.collide_point(*touch.pos):
            try:
                global num
                num-=1
                global answer_list
                global recent_array
                recent_array=answer_list[num]
            except:
                pass

    
class Answercanvas(Widget):
    def on_touch_down(self, touch):
        global recent_array
        if self.collide_point(*touch.pos):
            x=int((touch.x-46)/77)
            y=4-int((touch.y-67)/77)
            recent_array[y,x]=0
        dd=self.draw(recent_array)

            
    def Choosecolor(self,num):
        b=0
        g=0
        r=0
        if num==0:
            b=1
            g=1
            r=1
        elif num>0 and num<30:
            b=1
            g=0
            r=0
        elif num>30 and num<40:
            b=0
            g=1
            r=0
        return b,g,r
    
    def draw(self,array):
        with self.canvas:
            Color(1,1,1)
            Rectangle(pos=self.pos,size=self.size)
        for i in range(5):
            for ii in range(4):
                b,g,r=self.Choosecolor(array[i,ii])
                with self.canvas:
                    Color(b,g,r)
                    d=70
                    Rectangle(pos=(ii*80+45,(4-i)*80+65),size=(d,d))
                if ii<3:
                    if array[i,ii]==array[i,ii+1] and array[i,ii]!=31:
                        with self.canvas:
                            Color(b,g,r)
                            d=70
                            Rectangle(pos=(ii*80+85,(4-i)*80+65),size=(d,d))
                if i<4:
                    if array[i,ii]==array[i+1,ii] and array[i,ii]!=31:
                        with self.canvas:
                            Color(b,g,r)
                            d=70
                            Rectangle(pos=(ii*80+45,(4-i)*80+25),size=(d,d))
                if i<4 and ii<3:
                    if array[i,ii]==41 and array[i,ii]==array[i+1,ii+1]:
                        with self.canvas:
                            Color(b,g,r)
                            d=70
                            Rectangle(pos=(ii*80+85,(4-i)*80+25),size=(d,d))
                if array[i,ii]==1000:
                    with self.canvas:
                        Color(1,1,1)
                        Rectangle(pos=self.pos,size=self.size)
                    
        
    
            
        
class PictureSource(TextInput):
     pass
            

         

class HuaRongDaoApp(App):
    def build(self):
        def on_text(instance, value):
            print('The widget', instance, 'have:', value)
            global source
            source=value
            
        self.parent = Widget()
        parent=self.parent
        parent.size=800,600
        
        parent.add_widget(AsyncImage(source="background.png",size=[800,600]))
        
        with parent.canvas.before:
            Color(0, 1, 0, 1) # green; colors range from 0-1 instead of 0-255
            parent.rect = Rectangle(size=parent.size, pos=parent.pos)
        with parent.canvas.before:
            Color(1, 1, 1, 1)
            parent.rect2 = Rectangle(size=(20,parent.height), pos=(1/2*parent.width-10,0))
        
        parent.add_widget(Label(text='Answer Ground',
                              width=parent.width*0.4,
                              height=50,
                              font_size=50,
                              pos=(parent.width*0.05,parent.height-80)))
        parent.add_widget(Label(text='Option Ground',
                              width=parent.width*0.4,
                              height=50,
                              font_size=50,
                              pos=(parent.width*0.55+10,parent.height-80)))
        
        parent.add_widget(Label(text='Picture Source',
                              width=parent.width*0.4,
                              height=50,
                              font_size=30,
                              pos=(parent.width*0.55+10,parent.height-170)))
        
        
        parent.username=PictureSource()
        parent.username.multiline=False
        parent.add_widget(parent.username)
        parent.username.width=200
        parent.username.height=50
        parent.username.font_size=25
        parent.username.pos=((parent.width*0.5+10-200)*1/2+0.5*parent.width,parent.height*.65)
        parent.username.bind(text=on_text)
        
        parent.answerground=Answercanvas()
        parent.add_widget(parent.answerground)
        parent.answerground.size=(parent.width*0.4,parent.height*2/3)
        parent.answerground.pos=(parent.width*0.05,parent.height*1/10)
        with parent.answerground.canvas.before:
            Color(1, 1, 1, 1) # green; colors range from 0-1 instead of 0-255
            parent.answerground.rect = Rectangle(size=parent.answerground.size, pos=parent.answerground.pos)
        
        
        parent.precious=PreciousWidget()
        parent.precious.text='previous'
        parent.add_widget(parent.precious)
        parent.precious.pos=(parent.width*0.8,parent.height*3/8)
        
        parent.next=NextWidget()
        parent.next.text='next'
        parent.add_widget(parent.next)
        parent.next.pos=(parent.width*0.6,parent.height*3/8)
        
        parent.clear=ClearWidget()
        parent.clear.text='clear'
        parent.add_widget(parent.clear)
        parent.clear.pos=(parent.width*0.8,parent.height*1/8)
        
        parent.start=StartWidget()
        parent.start.text='start'
        parent.add_widget(parent.start)
        parent.start.pos=(parent.width*0.6,parent.height*1/8)
        
        parent.choose=ChooseWidget()
        parent.choose.text='choose'
        parent.add_widget(parent.choose)
        parent.choose.size=(100,50)
        parent.choose.pos=(parent.width*0.695,parent.height*0.56)
        
        
        
        #clearbtn = Button(text='Clear')
        #clearbtn.bind(on_release=self.clear_canvas)
        return parent










class pic_translate(object):
    import cv2
    import numpy as np
    import copy
    import time
    img=0
    final_array=[]
    def __init__(self,pic):
        self.img=cv2.imread(pic)
        self.final_array=self.get_position()
    
    def get_position(self):
        graylist=self.gray_pic()
        blurlist=self.blur_pic(graylist)
        cannylist=self.canny_pic(blurlist)
        img_line=self.line_pic(cannylist)
        allp_y,allp_x,ylen,xlen,ymin,xmin,ymax,xmax=self.point_pic(img_line)
        empty_y,empty_x=self.test(allp_y,allp_x,img_line,ylen,xlen)
        quartet,double_x,double_y=self.find_shape(empty_y,empty_x,ylen,xlen)
        finall_position=self.translate(quartet,double_x,double_y,ylen,xlen,ymin,xmin)
        return finall_position
        
        
    def gray_pic(self):
        gray_img=cv2.cvtColor(self.img,cv2.COLOR_BGR2GRAY)
        graylist=[gray_img]*5
        return graylist

    def blur_pic(self,gray_list):
        blurlist=gray_list
        kernel_blur=np.array([[0.04,0.04,0.04,0.04,0.04],
                              [0.04,0.04,0.04,0.04,0.04],
                              [0.04,0.04,0.04,0.04,0.04],
                              [0.04,0.04,0.04,0.04,0.04],
                              [0.04,0.04,0.04,0.04,0.04]])
        blurlist[0]=cv2.medianBlur(blurlist[0],3)
        blurlist[1]=cv2.medianBlur(blurlist[1],5)
        cv2.filter2D(blurlist[2],-1,kernel_blur,blurlist[2])
        blurlist[3] = cv2.GaussianBlur(blurlist[3],(5,5),0)
        blurlist[4] = cv2.GaussianBlur(blurlist[4],(3,3),0)
        return blurlist

    def canny_pic(self,blur_list):
        cannylist=blur_list
        for i in range(len(cannylist)):  
            cannylist[i]=cv2.Canny(cannylist[i],50,120)
        return cannylist
    
    def line_pic(self,canny_list,minLineLength=200,maxLineGap=10,f=20):
        pre_linelist=canny_list
        lines=[0]*len(pre_linelist)
        xmin=(1/30)*len(pre_linelist[0][0])
        xmax=(29/30)*len(pre_linelist[0][0])
        ymin=(1/30)*len(pre_linelist[0])
        ymax=(29/30)*len(pre_linelist[0])
        img_line=pre_linelist[0].copy()
        for x in range(len(img_line[0])):
            for y in range(len(img_line)):      
                if img_line[y,x]>0:
                    img_line[y,x]=0
        for i in range(len(pre_linelist)):     
            lines[i]=cv2.HoughLinesP(pre_linelist[i],1,np.pi/180,100,minLineLength,maxLineGap)
            if len(lines[i])<100:
                print('100')
                for i in lines[i]:
                    for x1,y1,x2,y2 in i:
                        if xmin<x1<xmax and xmin<x2<xmax and ymin<y1<ymax and ymin<y2<ymax:
                            x1,y1,x2,y2=x1//f*f+1,y1//f*f+1,x2//f*f+1,y2//f*f+1
                            cv2.line(img_line,(x1,y1),(x2,y2),255,2)
            else:
                print(len(lines[i]))
        return img_line
                    
    def point_pic(self,img_line):
        img_point=img_line
        dst=cv2.cornerHarris(img_point,2,31,0.04)
        dst_=dst.copy()
        dst_[dst>0.01*dst.max()]=1
        dst_[dst<0.01*dst.max()]=0
        py=[]
        px=[]
        for i in range(len(dst_)):
            if dst_[i].mean()!=0:
                for ii in range(len(dst_[0])):
                    if dst_[i][ii]==1:
                        py.append(i)
                        px.append(ii)
        py.sort()
        px.sort()
        for i in py:
            if py.count(i)>4:
                ymin=i
                break
        for i in px:
            if px.count(i)>4:
                xmin=i
                break   
        py.reverse()
        px.reverse()
        for i in py:
            if py.count(i)>4:
                ymax=i
                break
        for i in px:
            if px.count(i)>4:
                xmax=i
                break
            
        ylen=int((ymax-ymin)/5)
        xlen=int((xmax-xmin)/4)
        allp_y=[]
        for i in range(4):
            y=ymin+(1+i)*ylen
            for ii in range(4):
                x=xmin+(1/2)*xlen+ii*xlen
                allp_y.append([y,x])
        allp_x=[]
        for i in range(5):
            y=ymin+i*ylen+(1/2)*ylen
            for ii in range(3):
                x=xmin+xlen+ii*xlen
                allp_x.append([y,x])
        return allp_y,allp_x,ylen,xlen,ymin,xmin,ymax,xmax

    def test(self,allp_y_,allp_x_,img_line_,ylen_,xlen_):
        xlen=xlen_
        ylen=ylen_
        img=img_line_
        empty_y=[]
        empty_x=[]
        test_y=allp_y_
        test_x=allp_x_
        for i in test_y:
            y=int(i[0])
            x=int(i[1])
            flag=0
            count=0
            for ii in range(5):
                for iii in range(int(1/2*xlen)):
                    if img[y-2+ii][x-int(1/4*xlen)+iii]==255:
                        count=count+1
                        if count>=10:
                            flag=1
                            break
                if flag==1:
                    break
                else:
                    if ii==4:
                        empty_y.append([y,x])
        for i in test_x:
            y=int(i[0])
            x=int(i[1])
            flag=0
            count=0
            for ii in range(5):
                for iii in range(int(1/2*ylen)):
                    if img[y-int(1/4*ylen)+iii][x-2+ii]==255:
                        count=count+1
                        if count>=10:
                            flag=1
                            break
                if flag==1:
                    break
                else:
                    if ii==4:
                        empty_x.append([y,x])   
        empty_y.sort()
        empty_x.sort()
        return empty_y,empty_x

    def find_shape(self,empty_y_,empty_x_,ylen_,xlen_):
        xlen=xlen_
        ylen=ylen_
        empty_y=empty_y_
        empty_x=empty_x_
        quartet=[]
        double_y=empty_y.copy()
        double_x=empty_x.copy()
        for i in empty_y:
            for ii in range(10):
                for iii in range(10):
                    if [i[0]-1/2*ylen-5+ii,i[1]+1/2*xlen-5+iii] in empty_x and [i[0]+1/2*ylen-5+ii,i[1]+1/2*xlen-5+iii] in empty_x :
                        quartet.append(i)
                        double_y.remove(i)
                    elif [i[0]+1/2*ylen-5+ii,i[1]-1/2*xlen-5+iii] in empty_x:        
                        double_y.remove(i)
        for i in empty_x:
            for ii in range(10):
                for iii in range(10):
                    if ([i[0]+1/2*ylen-5+ii,i[1]-1/2*xlen-5+iii] in empty_y) or ([i[0]-1/2*ylen-5+ii,i[1]+1/2*xlen-5+iii] in empty_y):
                        if i in double_x:       
                            double_x.remove(i)
        return  quartet,double_x,double_y

    def translate(self,quartet_,double_x_,double_y_,ylen_,xlen_,ymin_,xmin_):
        xlen=xlen_
        ylen=ylen_
        ymin=ymin_
        xmin=xmin_
        #ymax=ymax_
        #xmax=xmax_
        quartet=quartet_
        double_x=double_x_
        double_y=double_y_                 
        finall_position=np.array([[31]*4]*5)
        d=20
        q=41
        for i in double_y:
            d=d+1
            finall_position[int((i[0]-1/2*ylen-ymin)//ylen),int((i[1]-xmin)//xlen)]=d
            finall_position[int((i[0]-1/2*ylen-ymin)//ylen+1),int((i[1]-xmin)//xlen)]=d
        for i in double_x:
            d=d+1
            finall_position[int((i[0]-ymin)//ylen),int((i[1]-1/2*xlen-xmin)//xlen)]=d
            finall_position[int((i[0]-ymin)//ylen),int((i[1]-1/2*xlen-xmin)//xlen+1)]=d
        for i in quartet:
            finall_position[int((i[0]-1/2*ylen-ymin)//ylen),int((i[1]-xmin)//xlen)]=q
            finall_position[int((i[0]-1/2*ylen-ymin)//ylen+1),int((i[1]-xmin)//xlen)]=q
            finall_position[int((i[0]-1/2*ylen-ymin)//ylen),int((i[1]-xmin)//xlen+1)]=q
            finall_position[int((i[0]-1/2*ylen-ymin)//ylen+1),int((i[1]-xmin)//xlen+1)]=q
        return finall_position   





class HRD(object):
    import time
    import pandas as pd
    import numpy as np
    import copy
    import pickle
    whole=[]
    ways=[]
    new_whole=[]
    answer=[]
    num=0
    a=True
    origin_way=[]
    findanswer=0
    t1=time.time()-time.time()
    def __init__(self,a):
        self.ways.append(a)
        self.whole.append(self.ways)
        self.findanswer=self.Find_answer()

                

        
    
    def Chooseways(self):
        self.new_whole=self.whole.copy()
        self.whole=[]
        for i in range(len(self.new_whole)):
            the_way=self.new_whole[i]
            array=self.new_whole[i][-1]
            road_loc=self.Roadloc(array)
            self.Move_1(array,the_way,road_loc)
            if road_loc[0][0]==road_loc[1][0]: 
                if abs(road_loc[0][1]-road_loc[1][1])==1:
                    self.Move_2(array,the_way,road_loc)
            elif road_loc[0][1]==road_loc[1][1]:
                if abs(road_loc[0][0]-road_loc[1][0])==1:
                    self.Move_3(array,the_way,road_loc)
    
    
    def Roadloc(self,a):
        road_loc=[[10,10],[10,10]]
        for i in range(5):
            for ii in range(4):
                if a[i][ii]==0:
                    if road_loc[0][0]==10:
                        road_loc[0]=[i,ii]
                    else:
                        road_loc[1]=[i,ii]
        return road_loc
    def Appendways(self,new,way,num):
        new_way=way.copy()
        new_way.append(new)
        self.whole.append(new_way)
        
    def Move_1(self,array,way,road):
        a=array
        the_ways=way
        road_loc=road
        for i in range(2):           
            if road_loc[i][0]>0:
                if a[road_loc[i][0]-1][road_loc[i][1]]==31:
                    new=a.copy()
                    new[road_loc[i][0]-1][road_loc[i][1]],new[road_loc[i][0]][road_loc[i][1]]=new[road_loc[i][0]][road_loc[i][1]],new[road_loc[i][0]-1][road_loc[i][1]]
                    self.Appendways(new,the_ways,1)
                elif road_loc[i][0]>1:
                    if (a[road_loc[i][0]-1][road_loc[i][1]]==a[road_loc[i][0]-2][road_loc[i][1]])and(a[road_loc[i][0]-1][road_loc[i][1]]!=41):
                        new=a.copy()
                        new[road_loc[i][0]-2][road_loc[i][1]],new[road_loc[i][0]][road_loc[i][1]]=new[road_loc[i][0]][road_loc[i][1]],new[road_loc[i][0]-2][road_loc[i][1]]
                        self.Appendways(new,the_ways,2)
            if road_loc[i][0]<4:
                if a[road_loc[i][0]+1][road_loc[i][1]]==31:
                    new=a.copy()
                    new[road_loc[i][0]+1][road_loc[i][1]],new[road_loc[i][0]][road_loc[i][1]]=new[road_loc[i][0]][road_loc[i][1]],new[road_loc[i][0]+1][road_loc[i][1]]
                    self.Appendways(new,the_ways,3)
                elif road_loc[i][0]<3:
                    if (a[road_loc[i][0]+1][road_loc[i][1]]==a[road_loc[i][0]+2][road_loc[i][1]])and(a[road_loc[i][0]-1][road_loc[i][1]]!=41):
                        new=a.copy()
                        new[road_loc[i][0]+2][road_loc[i][1]],new[road_loc[i][0]][road_loc[i][1]]=new[road_loc[i][0]][road_loc[i][1]],new[road_loc[i][0]+2][road_loc[i][1]]
                        self.Appendways(new,the_ways,4)
            if road_loc[i][1]>0:
                if a[road_loc[i][0]][road_loc[i][1]-1]==31:
                    new=a.copy()
                    new[road_loc[i][0]][road_loc[i][1]-1],new[road_loc[i][0]][road_loc[i][1]]=new[road_loc[i][0]][road_loc[i][1]],new[road_loc[i][0]][road_loc[i][1]-1]
                    self.Appendways(new,the_ways,5)
                elif road_loc[i][1]>1:
                    if (a[road_loc[i][0]][road_loc[i][1]-1]==a[road_loc[i][0]][road_loc[i][1]-2])and(a[road_loc[i][0]][road_loc[i][1]-1]!=41):
                        new=a.copy()
                        new[road_loc[i][0]][road_loc[i][1]-2],new[road_loc[i][0]][road_loc[i][1]]=new[road_loc[i][0]][road_loc[i][1]],new[road_loc[i][0]][road_loc[i][1]-2]
                        self.Appendways(new,the_ways,6)
            if road_loc[i][1]<3:
                if a[road_loc[i][0]][road_loc[i][1]+1]==31:
                    new=a.copy()
                    new[road_loc[i][0]][road_loc[i][1]+1],new[road_loc[i][0]][road_loc[i][1]]=new[road_loc[i][0]][road_loc[i][1]],new[road_loc[i][0]][road_loc[i][1]+1]
                    self.Appendways(new,the_ways,7)
                elif road_loc[i][1]<2:
                    if (a[road_loc[i][0]][road_loc[i][1]+1]==a[road_loc[i][0]][road_loc[i][1]+2])and(a[road_loc[i][0]][road_loc[i][1]+1]!=41):
                        new=a.copy()
                        new[road_loc[i][0]][road_loc[i][1]+2],new[road_loc[i][0]][road_loc[i][1]]=new[road_loc[i][0]][road_loc[i][1]],new[road_loc[i][0]][road_loc[i][1]+2]
                        self.Appendways(new,the_ways,8)
    def Move_2(self,array,way,road):
        a=array
        the_ways=way
        road_loc=road
        if road_loc[0][0]>0:
            if a[road_loc[0][0]-1][road_loc[0][1]]==a[road_loc[1][0]-1][road_loc[1][1]]:
                if a[road_loc[0][0]-1][road_loc[0][1]]!=31:
                    if a[road_loc[0][0]-1][road_loc[0][1]]!=41:
                        new=a.copy()
                        new[road_loc[0][0]-1][road_loc[0][1]],new[road_loc[0][0]][road_loc[0][1]]=new[road_loc[0][0]][road_loc[0][1]],new[road_loc[0][0]-1][road_loc[0][1]]
                        new[road_loc[1][0]-1][road_loc[1][1]],new[road_loc[1][0]][road_loc[1][1]]=new[road_loc[1][0]][road_loc[1][1]],new[road_loc[1][0]-1][road_loc[1][1]]
                        self.Appendways(new,the_ways,9)
                    else:
                        new=a.copy()
                        new[road_loc[0][0]-2][road_loc[0][1]],new[road_loc[0][0]][road_loc[0][1]]=new[road_loc[0][0]][road_loc[0][1]],new[road_loc[0][0]-2][road_loc[0][1]]
                        new[road_loc[1][0]-2][road_loc[1][1]],new[road_loc[1][0]][road_loc[1][1]]=new[road_loc[1][0]][road_loc[1][1]],new[road_loc[1][0]-2][road_loc[1][1]]
                        self.Appendways(new,the_ways,10)
        if road_loc[0][0]<4:
            if a[road_loc[0][0]+1][road_loc[0][1]]==a[road_loc[1][0]+1][road_loc[1][1]]:
                if a[road_loc[0][0]+1][road_loc[0][1]]!=31:
                    if a[road_loc[0][0]+1][road_loc[0][1]]!=41:
                        new=a.copy()
                        new[road_loc[0][0]+1][road_loc[0][1]],new[road_loc[0][0]][road_loc[0][1]]=new[road_loc[0][0]][road_loc[0][1]],new[road_loc[0][0]+1][road_loc[0][1]]
                        new[road_loc[1][0]+1][road_loc[1][1]],new[road_loc[1][0]][road_loc[1][1]]=new[road_loc[1][0]][road_loc[1][1]],new[road_loc[1][0]+1][road_loc[1][1]]
                        self.Appendways(new,the_ways,11)
                    else:
                        new=a.copy()
                        new[road_loc[0][0]+2][road_loc[0][1]],new[road_loc[0][0]][road_loc[0][1]]=new[road_loc[0][0]][road_loc[0][1]],new[road_loc[0][0]+2][road_loc[0][1]]
                        new[road_loc[1][0]+2][road_loc[1][1]],new[road_loc[1][0]][road_loc[1][1]]=new[road_loc[1][0]][road_loc[1][1]],new[road_loc[1][0]+2][road_loc[1][1]]
                        self.Appendways(new,the_ways,12)
    def Move_3(self,array,way,road):
        a=array
        the_ways=way
        road_loc=road
        if road_loc[0][1]>0:
            if a[road_loc[0][0]][road_loc[0][1]-1]==a[road_loc[1][0]][road_loc[1][1]-1]:
                if a[road_loc[0][0]][road_loc[0][1]-1]!=31:
                    if a[road_loc[0][0]][road_loc[0][1]-1]!=41:
                        new=a.copy()
                        new[road_loc[0][0]][road_loc[0][1]-1],new[road_loc[0][0]][road_loc[0][1]]=new[road_loc[0][0]][road_loc[0][1]],new[road_loc[0][0]][road_loc[0][1]-1]
                        new[road_loc[1][0]][road_loc[1][1]-1],new[road_loc[1][0]][road_loc[1][1]]=new[road_loc[1][0]][road_loc[1][1]],new[road_loc[1][0]][road_loc[1][1]-1]
                        self.Appendways(new,the_ways,13)
                    else:
                        new=a.copy()
                        new[road_loc[0][0]][road_loc[0][1]-2],new[road_loc[0][0]][road_loc[0][1]]=new[road_loc[0][0]][road_loc[0][1]],new[road_loc[0][0]][road_loc[0][1]-2]
                        new[road_loc[1][0]][road_loc[1][1]-2],new[road_loc[1][0]][road_loc[1][1]]=new[road_loc[1][0]][road_loc[1][1]],new[road_loc[1][0]][road_loc[1][1]-2]
                        self.Appendways(new,the_ways,14)
        if road_loc[0][1]<3:
            if a[road_loc[0][0]][road_loc[0][1]+1]==a[road_loc[1][0]][road_loc[1][1]+1]:
                if a[road_loc[0][0]][road_loc[0][1]+1]!=31:
                    if a[road_loc[0][0]][road_loc[0][1]+1]!=41:
                        new=a.copy()
                        new[road_loc[0][0]][road_loc[0][1]+1],new[road_loc[0][0]][road_loc[0][1]]=new[road_loc[0][0]][road_loc[0][1]],new[road_loc[0][0]][road_loc[0][1]+1]
                        new[road_loc[1][0]][road_loc[1][1]+1],new[road_loc[1][0]][road_loc[1][1]]=new[road_loc[1][0]][road_loc[1][1]],new[road_loc[1][0]][road_loc[1][1]+1]
                        self.Appendways(new,the_ways,15)
                    else:
                        new=a.copy()
                        new[road_loc[0][0]][road_loc[0][1]+2],new[road_loc[0][0]][road_loc[0][1]]=new[road_loc[0][0]][road_loc[0][1]],new[road_loc[0][0]][road_loc[0][1]+2]
                        new[road_loc[1][0]][road_loc[1][1]+2],new[road_loc[1][0]][road_loc[1][1]]=new[road_loc[1][0]][road_loc[1][1]],new[road_loc[1][0]][road_loc[1][1]+2]
                        self.Appendways(new,the_ways,16)
    
    def Del_repeat(self):
        l=len(self.whole)
        for i in range(-l+1,1):
            array=self.whole[-i][-1]
            c=self.Delete(array,-i)
            if c:
                tarray=self.Trans(array)
                self.Delete(tarray,-i)
    
    
    def Delete(self,test ,num):
        if len(self.whole[0])>75:       
            for i in range(len(self.whole)):
                for ii in range(len(self.whole[0])-10,len(self.whole[0])):
                    if (abs(test-self.whole[i][ii])<np.array([[7]*4]*5)).all()==(np.array([[True]*4]*5)).all():
                        if not((i==num)and(ii==(len(self.whole[0])-1))):
                            del self.whole[num]
                            return False
            return True 
        if len(self.whole[0])>60:       
            for i in range(len(self.whole)):
                for ii in range(len(self.whole[0])-15,len(self.whole[0])):
                    if (abs(test-self.whole[i][ii])<np.array([[7]*4]*5)).all()==(np.array([[True]*4]*5)).all():
                        if not((i==num)and(ii==(len(self.whole[0])-1))):
                            del self.whole[num]
                            return False
            return True
        elif len(self.whole[0])>40:       
            for i in range(len(self.whole)):
                for ii in range(len(self.whole[0])-25,len(self.whole[0])):
                    if (abs(test-self.whole[i][ii])<np.array([[7]*4]*5)).all()==(np.array([[True]*4]*5)).all():
                        if not((i==num)and(ii==(len(self.whole[0])-1))):
                            del self.whole[num]
                            return False
            return True
        else:
            for i in range(len(self.whole)):
                for ii in range(len(self.whole[0])):
                    if (abs(test-self.whole[i][ii])<np.array([[7]*4]*5)).all()==(np.array([[True]*4]*5)).all():
                        if not((i==num)and(ii==(len(self.whole[0])-1))):
                            del self.whole[num]
                            return False
            return True
                     
    def Trans(self,array):
        a=array.T
        a[0],a[1],a[2],a[3]=a[3].copy(),a[2].copy(),a[1].copy(),a[0].copy()
        b=a.T.copy()
        return b
    
    def Evalute(self):
        for i in range(len(self.whole)):
            for ii in range(2):   
                for iii in range(len(self.origin_way[0])):
                    if (abs(self.origin_way[ii][iii]-self.whole[i][-1])<np.array([[7]*4]*5)).all()==(np.array([[True]*4]*5)).all():
                        '''flag=False
                        for k in range(len(self.whole)-1):
                            if self.whole[k+1][0][0]!=31 and self.whole[k][0][0]!=self.whole[k+1][0][0] and self.whole[k][0][3]==self.whole[k+1][0][0]:
                                self.whole[k+1]=self.Trans(self.whole[k+1])
                                if k==len(self.whole)-2:
                                    flag=True
                            elif self.whole[k+1][2][0]!=31 and self.whole[k][2][0]!=self.whole[k+1][2][0] and self.whole[k][2][3]==self.whole[k+1][2][0]:
                                self.whole[k+1]=self.Trans(self.whole[k+1])
                                if k==len(self.whole)-2:
                                    flag=True
                            elif self.whole[k+1][2][3]!=31 and self.whole[k][2][3]!=self.whole[k+1][2][3] and self.whole[k][2][0]==self.whole[k+1][2][3]:
                                self.whole[i+1]=self.Trans(self.whole[i+1])
                                if k==len(self.whole)-2:
                                    flag=True
                            elif self.whole[k+1][4][0]!=31 and self.whole[k][4][0]!=self.whole[k+1][4][0] and self.whole[k][4][3]==self.whole[k+1][4][0]:
                                self.whole[k+1]=self.Trans(self.whole[k+1])
                                if k==len(self.whole)-2:
                                    flag=True
                        if flag:
                            for k in range(len(self.origin_way[ii][iii+1:])-1): 
                                if self.origin_way[ii][iii+1:][k+1][0][0]!=31 and self.origin_way[ii][iii+1:][k][0][0]!=self.origin_way[ii][iii+1:][k+1][0][0] and self.origin_way[ii][iii+1:][k][0][3]==self.origin_way[ii][iii+1:][k+1][0][0]:
                                    self.origin_way[ii][iii+1:][k+1]=self.Trans(self.origin_way[ii][iii+1:][k+1])
                                elif self.origin_way[ii][iii+1:][k+1][2][0]!=31 and self.origin_way[ii][iii+1:][k][2][0]!=self.origin_way[ii][iii+1:][k+1][2][0] and self.origin_way[ii][iii+1:][k][2][3]==self.origin_way[ii][iii+1:][k+1][2][0]:
                                    self.origin_way[ii][iii+1:][k+1]=self.Trans(self.origin_way[ii][iii+1:][k+1])
                                elif self.origin_way[ii][iii+1:][k+1][2][3]!=31 and self.origin_way[ii][iii+1:][k][2][3]!=self.origin_way[ii][iii+1:][k+1][2][3] and self.origin_way[ii][iii+1:][k][2][0]==self.origin_way[ii][iii+1:][k+1][2][3]:
                                    self.origin_way[ii][iii+1:][i+1]=self.Trans(self.origin_way[ii][iii+1:][i+1])
                                elif self.origin_way[ii][iii+1:][k+1][4][0]!=31 and self.origin_way[ii][iii+1:][k][4][0]!=self.origin_way[ii][iii+1:][k+1][4][0] and self.origin_way[ii][iii+1:][k][4][3]==self.origin_way[ii][iii+1:][k+1][4][0]:
                                    self.origin_way[ii][iii+1:][k+1]=self.Trans(self.origin_way[ii][iii+1:][k+1])'''
                        self.answer=self.whole[i]+self.origin_way[ii][iii+1:]
                        return False
        return True
    
    def Find_answer(self):
        pkl_file = open('HRD_answer.pkl', 'rb')
        self.origin_way=pickle.load(pkl_file)
        pkl_file.close()
        self.Runing()
        
    def Runing(self):
        t=time.time()
        self.Chooseways()
        self.Del_repeat()
        self.t1=self.t1+time.time()-t
        try:
            print('步数:',len(self.whole[0]),'走法: ',len(self.whole),'阶段用时:',time.time()-t,'总用时:',self.t1)
        except:
            self.a=False
            print('无解')
        if self.a:
            if self.Evalute():
                self.Runing()
            else:
                print('success')
                return 0
    def Evalute_whole(self):
        for i in range(len(self.whole)):
            for ii in range(len(self.whole[0])):
                if(self.whole[i][ii])[4][1]==41:
                    if (self.whole[i][ii])[4][2]==41:
                        print([i,ii,'success'])
        print('fail')
    def Runing_test(self,num):
        for i in range(num):
            self.Chooseways()
            self.Del_repeat()
        self.Evalute_whole()
    

    
    
    
    
if __name__ == '__main__':
    HuaRongDaoApp().run()
