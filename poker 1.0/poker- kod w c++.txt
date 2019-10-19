#include<SFML/Graphics.hpp>
#include<cmath>
#include<ctime>
#include<iostream>
#include<string>
#include<fstream>
#include<cstdlib>
#include<cstdio>
#include<vector>
#include<algorithm>
#include<windows.h>
using namespace std;
using namespace sf;

struct Card
{
    int ID;
    int rank,suit;
    Sprite sprCard;
};
bool operator==(const Card &a,const Card &b)
{
    return a.rank==b.rank;
}
bool operator!=(const Card &a,const Card &b)
{
    return !(a.rank==b.rank);
}
bool operator<(const Card &a,const Card &b)
{
    return a.rank<b.rank;
}
bool operator>(const Card &a,const Card &b)
{
    return a.rank>b.rank;
}
bool operator>=(const Card &a,const Card &b)
{
    return a.rank>=b.rank;
}
bool operator<=(const Card &a,const Card &b)
{
    return a.rank<=b.rank;
}

struct Player
{
    string ID;
    int nr;
    int totalMoney,moneyIn,sumOfAllIn,moneyReceived;
    bool inRound,inGame;
    Card card1,card2;
    void goIn(int money)
    {
        moneyIn+=money;
        totalMoney-=money;
    }
    void goAllIn()
    {
        moneyIn+=totalMoney;
        totalMoney=0;
        isAllIn=true;
    }
    bool isAllIn;
    Texture tex0,tex1,tex2;
    Sprite sprAvatar,sprMoneyIn[5],sprTotalMoney[5];
};

vector<Player> player(7);
vector<Card> deck(52);
vector<Card> cardsOnTable(5);
Event event;
Clock clocky;
Card emptyCard;
Texture texNewGame[2],texGameOver,texCards,texBackground,texRaise[3],texNumbers[11],texMoneyIcon,texCheck[3],texCall[3],texFold[3],tex100[3],tex200[3],tex500[3],tex1000[3],tex2000[3],texHalfPot[3],texPot[3],texAllIn[3];
Sprite sprNewGame,sprMoneyOnTable[5],sprGameOver,sprBackground,sprRaise,sprCheckOrCall,sprFold,sprCards[52],sprBorder,spr100,spr200,spr500,spr1000,spr2000,sprHalfPot,sprPot,sprAllIn;
bool drawRaiseButtons=0;
bool canIGo(const int money)
{
    return (player[0].totalMoney>=money);
}
///m=: start=0, flop=1, turn=2, river=3, 3 pierwsze = 4, 4 pierwsze = 5

int highcard(Card Playertable[7], int m)
{
    int x,y;
    if (m==0)
    {
        y=2;
        x=Playertable[0].rank;
        for(int i=0; i<y-1; i++)
        {
            if(Playertable[i].rank<Playertable[i+1].rank)
            {
                x=Playertable[i+1].rank;
            }
        }
    }
    if (m==1)
    {
        y=5;
        x=Playertable[0].rank;
        for(int i=0; i<y-1; i++)
        {
            if(Playertable[i].rank<Playertable[i+1].rank)
            {
                x=Playertable[i+1].rank;
            }
        }
    }
    if (m==2)
    {
        y=6;
        x=Playertable[0].rank;
        for(int i=0; i<y-1; i++)
        {
            if(Playertable[i].rank<Playertable[i+1].rank)
            {
                x=Playertable[i+1].rank;
            }
        }
    }
    if (m==3)
    {
        y=7;
        x=Playertable[0].rank;
        for(int i=0; i<y-1; i++)
        {
            if(Playertable[i].rank<Playertable[i+1].rank)
            {
                x=Playertable[i+1].rank;
            }
        }
    }
    return x;
}
bool pairr(Card Playertable[7], int m)
{
    int x,y;
    if (m==0)
    {
        y=2;
        if(Playertable[0].rank==Playertable[1].rank)
        {
            return true;
        }
    }
    if (m==1)
    {
        y=5;
        for(int i=0; i<y-1; i++)
        {
            for(int j=1; j<y-i; j++)
            {
                if(Playertable[i].rank==Playertable[i+j].rank)
                {
                    return true;
                }
            }
        }
    }
    if (m==2)
    {
        y=6;
        for(int i=0; i<y-1; i++)
        {
            for(int j=1; j<y-i; j++)
            {
                if(Playertable[i].rank==Playertable[i+j].rank)
                {
                    return true;
                }
            }
        }
    }
    if (m==3)
    {
        y=7;
        for(int i=0; i<y-1; i++)
        {
            for(int j=1; j<y-i; j++)
            {
                if(Playertable[i].rank==Playertable[i+j].rank)
                {
                    return true;
                }
            }
        }
    }
    if(m==4){
        y=3;
        for(int i=0; i<y-1; i++)
        {
            for(int j=1; j<y-i; j++)
            {
                if(Playertable[i].rank==Playertable[i+j].rank)
                {
                    return true;
                }
            }
        }
    }
    if(m==5){
        y=4;
        for(int i=0; i<y-1; i++)
        {
            for(int j=1; j<y-i; j++)
            {
                if(Playertable[i].rank==Playertable[i+j].rank)
                {
                    return true;
                }
            }
        }

    }
    return false;
}
bool twopair(Card Playertable[7], int m)
{
    int x,y,b;
    if (m==0)
    {
        y=2;
    }
    if (m==1)
    {
        y=5;
        for(int i=0; i<y-1; i++)
        {
            for(int j=1; j<y-i; j++)
            {
                if(Playertable[i].rank==Playertable[i+j].rank)
                {
                    b=Playertable[i].rank;
                    break;
                }
            }
        }
        for(int i=0; i<y-1; i++)
        {
            if (Playertable[i].rank!=b)
            {
                for(int j=1; j<y-i; j++)
                {
                    if(Playertable[i].rank==Playertable[i+j].rank)
                    {
                        return true;
                    }
                }
            }
        }
    }
    if (m==2)
    {
        y=6;
        for(int i=0; i<y-1; i++)
        {
            for(int j=1; j<y-i; j++)
            {
                if(Playertable[i].rank==Playertable[i+j].rank)
                {
                    b=Playertable[i].rank;
                    break;
                }
            }
        }
        for(int i=0; i<y-1; i++)
        {
            if (Playertable[i].rank!=b)
            {
                for(int j=1; j<y-i; j++)
                {
                    if(Playertable[i].rank==Playertable[i+j].rank)
                    {
                        return true;
                    }
                }
            }
        }
    }
    if (m==3)
    {
        y=7;
        for(int i=0; i<y-1; i++)
        {
            for(int j=1; j<y-i; j++)
            {
                if(Playertable[i].rank==Playertable[i+j].rank)
                {
                    b=Playertable[i].rank;
                    break;
                }
            }
        }
        for(int i=0; i<y-1; i++)
        {
            if (Playertable[i].rank!=b)
            {
                for(int j=1; j<y-i; j++)
                {
                    if(Playertable[i].rank==Playertable[i+j].rank)
                    {
                        return true;
                    }
                }
            }
        }
    }
    if(m==4){
        y=3;
        return false;

    }
    if(m==5){
        y=4;
        for(int i=0; i<y-1; i++)
        {
            for(int j=1; j<y-i; j++)
            {
                if(Playertable[i].rank==Playertable[i+j].rank)
                {
                    b=Playertable[i].rank;
                    break;
                }
            }
        }
        for(int i=0; i<y-1; i++)
        {
            if (Playertable[i].rank!=b)
            {
                for(int j=1; j<y-i; j++)
                {
                    if(Playertable[i].rank==Playertable[i+j].rank)
                    {
                        return true;
                    }
                }
            }
        }
    }
    return false;
}
bool three(Card Playertable[7], int m)
{
    int x,y;
    if (m==0)
    {
        y=2;
    }
    if (m==1)
    {
        y=5;
        for(int i=0; i<y-2; i++)
        {
            for(int j=1; j<y-i-1; j++)
            {
                if(Playertable[i].rank==Playertable[i+j].rank)
                {
                    for(int k=j+1; k<y-i; k++)
                    {
                        if(Playertable[i].rank==Playertable[i+k].rank)
                        {
                            return true;
                        }
                    }
                }
            }
        }
    }
    if (m==2)
    {
        y=6;
        for(int i=0; i<y-2; i++)
        {
            for(int j=1; j<y-i-1; j++)
            {
                if(Playertable[i].rank==Playertable[i+j].rank)
                {
                    for(int k=j+1; k<y-i; k++)
                    {
                        if(Playertable[i].rank==Playertable[i+k].rank)
                        {
                            return true;
                        }
                    }
                }
            }
        }
    }
    if (m==3)
    {
        y=7;
        for(int i=0; i<y-2; i++)
        {
            for(int j=1; j<y-i-1; j++)
            {
                if(Playertable[i].rank==Playertable[i+j].rank)
                {
                    for(int k=j+1; k<y-i; k++)
                    {
                        if(Playertable[i].rank==Playertable[i+k].rank)
                        {
                            return true;
                        }
                    }
                }
            }
        }
    }
    if(m==4){
        y=3;
        for(int i=0; i<y-2; i++)
        {
            for(int j=1; j<y-i-1; j++)
            {
                if(Playertable[i].rank==Playertable[i+j].rank)
                {
                    for(int k=j+1; k<y-i; k++)
                    {
                        if(Playertable[i].rank==Playertable[i+k].rank)
                        {
                            return true;
                        }
                    }
                }
            }
        }
    }
    if(m==5){
        y=4;
        for(int i=0; i<y-2; i++)
        {
            for(int j=1; j<y-i-1; j++)
            {
                if(Playertable[i].rank==Playertable[i+j].rank)
                {
                    for(int k=j+1; k<y-i; k++)
                    {
                        if(Playertable[i].rank==Playertable[i+k].rank)
                        {
                            return true;
                        }
                    }
                }
            }
        }
    }
    return false;
}
bool four(Card Playertable[7], int m)
{
    int x,y;
    if (m==0)
    {
        y=2;
    }
    if (m==1)
    {
        y=5;
        for(int i=0; i<y-3; i++)
    {
        for(int j=1; j<y-i-2; j++)
        {
            if(Playertable[i].rank==Playertable[i+j].rank)
            {
                for(int k=j+1; k<y-i-1; k++)
                {
                    if(Playertable[i].rank==Playertable[i+k].rank)
                    {
                        for(int h=k+1; h<y-i; h++)
                        {
                            if(Playertable[i].rank==Playertable[i+h].rank)
                            {
                                return true;
                            }
                        }
                    }
                }
            }
        }
    }
    }
    if (m==2)
    {
        y==6;
        for(int i=0; i<y-3; i++)
    {
        for(int j=1; j<y-i-2; j++)
        {
            if(Playertable[i].rank==Playertable[i+j].rank)
            {
                for(int k=j+1; k<y-i-1; k++)
                {
                    if(Playertable[i].rank==Playertable[i+k].rank)
                    {
                        for(int h=k+1; h<y-i; h++)
                        {
                            if(Playertable[i].rank==Playertable[i+h].rank)
                            {
                                return true;
                            }
                        }
                    }
                }
            }
        }
    }
    }
    if (m==3)
    {
        y=7;
        for(int i=0; i<y-3; i++)
    {
        for(int j=1; j<y-i-2; j++)
        {
            if(Playertable[i].rank==Playertable[i+j].rank)
            {
                for(int k=j+1; k<y-i-1; k++)
                {
                    if(Playertable[i].rank==Playertable[i+k].rank)
                    {
                        for(int h=k+1; h<y-i; h++)
                        {
                            if(Playertable[i].rank==Playertable[i+h].rank)
                            {
                                return true;
                            }
                        }
                    }
                }
            }
        }
    }
    }
    if(m==4){
        return false;
    }
    if(m==5){
        y=4;
        for(int i=0; i<y-3; i++)
    {
        for(int j=1; j<y-i-2; j++)
        {
            if(Playertable[i].rank==Playertable[i+j].rank)
            {
                for(int k=j+1; k<y-i-1; k++)
                {
                    if(Playertable[i].rank==Playertable[i+k].rank)
                    {
                        for(int h=k+1; h<y-i; h++)
                        {
                            if(Playertable[i].rank==Playertable[i+h].rank)
                            {
                                return true;
                            }
                        }
                    }
                }
            }
        }
    }

    }
    return false;
}
bool straight(Card Playertable[7], int m)
{
    int x,y,z;
    if (m==0)
    {
        y=2;
        return false;
    }
    if (m==1)
    {
        y=5;
        int tab[y];
        for(int i=0; i<y; i++)
        {
            tab[i]=Playertable[i].rank;
        }
        for (int i=0; i<y; i++)
        {
            for(int j=0; j<y-1; j++)
            {
                if(tab[j]>tab[j+1])
                {
                    z=tab[j+1];
                    tab[j+1]=tab[j];
                    tab[j]=z;
                }
            }
        }
        if(tab[0]==tab[1]-1 and tab[1]-1==tab[2]-2 and tab[2]-2==tab[3]-3 and tab[3]-3==tab[4]-4)
        {
            return true;
        }
    }
    if (m==2)
    {
        y=6;
        int tab[y];
        for(int i=0; i<y; i++)
        {
            tab[i]=Playertable[i].rank;
        }
        for (int i=0; i<y; i++)
        {
            for(int j=0; j<y-1; j++)
            {
                if(tab[j]>tab[j+1])
                {
                    z=tab[j+1];
                    tab[j+1]=tab[j];
                    tab[j]=z;
                }
            }
        }
        if((tab[0]==tab[1]-1 and tab[1]-1==tab[2]-2 and tab[2]-2==tab[3]-3 and tab[3]-3==tab[4]-4) or (tab[1]==tab[2]-1 and tab[2]-1==tab[3]-2 and tab[3]-2==tab[4]-3 and tab[4]-3==tab[5]-4))
        {
            return true;
        }
    }
    if (m==3)
    {
        y=7;
        int tab[y];
        for(int i=0; i<y; i++)
        {
            tab[i]=Playertable[i].rank;
        }
        for (int i=0; i<y; i++)
        {
            for(int j=0; j<y-1; j++)
            {
                if(tab[j]>tab[j+1])
                {
                    z=tab[j+1];
                    tab[j+1]=tab[j];
                    tab[j]=z;
                }
            }
        }
        if((tab[0]==tab[1]-1 and tab[1]-1==tab[2]-2 and tab[2]-2==tab[3]-3 and tab[3]-3==tab[4]-4) or (tab[1]==tab[2]-1 and tab[2]-1==tab[3]-2 and tab[3]-2==tab[4]-3 and tab[4]-3==tab[5]-4) or (tab[2]==tab[3]-1 and tab[3]-1==tab[4]-2 and tab[4]-2==tab[5]-3 and tab[5]-3==tab[6]-4))
        {
            return true;
        }
    }
    if(m==4){
        return false;
    }
    if(m==5){
        return false;
    }
    return false;

}
bool full(Card Playertable[7], int m)
{
    if(three(Playertable, m))
    {
        if(twopair(Playertable, m))
        {
            return true;
        }
    }
    return false;
}
bool flush(Card Playertable[7], int m)
{
    int a=0,b=0,c=0,d=0,y;
    if (m==0)
    {
        y=2;
        return false;
    }
    if (m==1)
    {
        y=5;
        for(int i=0; i<y; i++)
        {
            if(Playertable[i].suit==0)
            {
                a++;
            }
            if(Playertable[i].suit==1)
            {
                b++;
            }
            if(Playertable[i].suit==2)
            {
                c++;
            }
            if(Playertable[i].suit==3)
            {
                d++;
            }

        }
        if(a==5 or b==5 or c==5 or d==5)
        {
            return true;
        }
    }
    if (m==2)
    {
        y=6;
        for(int i=0; i<y; i++)
        {
            if(Playertable[i].suit==0)
            {
                a++;
            }
            if(Playertable[i].suit==1)
            {
                b++;
            }
            if(Playertable[i].suit==2)
            {
                c++;
            }
            if(Playertable[i].suit==3)
            {
                d++;
            }

        }
        if(a==5 or b==5 or c==5 or d==5)
        {
            return true;
        }
    }
    if (m==3)
    {
        y=7;
        for(int i=0; i<y; i++)
        {
            if(Playertable[i].suit==0)
            {
                a++;
            }
            if(Playertable[i].suit==1)
            {
                b++;
            }
            if(Playertable[i].suit==2)
            {
                c++;
            }
            if(Playertable[i].suit==3)
            {
                d++;
            }

        }
        if(a==5 or b==5 or c==5 or d==5)
        {
            return true;
        }

    }
    if(m==4){
        return false;
    }
    if(m==5){
        return false;
    }
    return false;
}
bool fourtoflush(Card Playertable[7], int m)
{
    int a=0,b=0,c=0,d=0,y;
    if (m==0)
    {
        y=2;
        return false;
    }
    if (m==1)
    {
        y=5;
        for(int i=0; i<y; i++)
        {
            if(Playertable[i].suit==0)
            {
                a++;
            }
            if(Playertable[i].suit==1)
            {
                b++;
            }
            if(Playertable[i].suit==2)
            {
                c++;
            }
            if(Playertable[i].suit==3)
            {
                d++;
            }

        }
        if(a==4 or b==4 or c==4 or d==4)
        {
            return true;
        }
    }
    if (m==2)
    {
        y=6;
        for(int i=0; i<y; i++)
        {
            if(Playertable[i].suit==0)
            {
                a++;
            }
            if(Playertable[i].suit==1)
            {
                b++;
            }
            if(Playertable[i].suit==2)
            {
                c++;
            }
            if(Playertable[i].suit==3)
            {
                d++;
            }

        }
        if(a==4 or b==4 or c==4 or d==4)
        {
            return true;
        }
    }
    if (m==3)
    {
        y=7;
        for(int i=0; i<y; i++)
        {
            if(Playertable[i].suit==0)
            {
                a++;
            }
            if(Playertable[i].suit==1)
            {
                b++;
            }
            if(Playertable[i].suit==2)
            {
                c++;
            }
            if(Playertable[i].suit==3)
            {
                d++;
            }

        }
        if(a==4 or b==4 or c==4 or d==4)
        {
            return true;
        }
    }
    if(m==4){
        return false;
    }
    if(m==5){
        y=4;
        y=7;
        for(int i=0; i<y; i++)
        {
            if(Playertable[i].suit==0)
            {
                a++;
            }
            if(Playertable[i].suit==1)
            {
                b++;
            }
            if(Playertable[i].suit==2)
            {
                c++;
            }
            if(Playertable[i].suit==3)
            {
                d++;
            }

        }
        if(a==4 or b==4 or c==4 or d==4)
        {
            return true;
        }
    }

    return false;
}
bool threetoflush(Card Playertable[7], int m)
{
    int a=0,b=0,c=0,d=0,y;
    if (m==0)
    {
        y=2;
        return false;
    }
    if (m==1)
    {
        y=5;
        for(int i=0; i<y; i++)
        {
            if(Playertable[i].suit==0)
            {
                a++;
            }
            if(Playertable[i].suit==1)
            {
                b++;
            }
            if(Playertable[i].suit==2)
            {
                c++;
            }
            if(Playertable[i].suit==3)
            {
                d++;
            }

        }
        if(a==3 or b==3 or c==3 or d==3)
        {
            return true;
        }
    }
    if (m==2)
    {
        y=6;
        for(int i=0; i<y; i++)
        {
            if(Playertable[i].suit==0)
            {
                a++;
            }
            if(Playertable[i].suit==1)
            {
                b++;
            }
            if(Playertable[i].suit==2)
            {
                c++;
            }
            if(Playertable[i].suit==3)
            {
                d++;
            }

        }
        if(a==3 or b==3 or c==3 or d==3)
        {
            return true;
        }
    }
    if (m==3)
    {
        y=7;
        for(int i=0; i<y; i++)
        {
            if(Playertable[i].suit==0)
            {
                a++;
            }
            if(Playertable[i].suit==1)
            {
                b++;
            }
            if(Playertable[i].suit==2)
            {
                c++;
            }
            if(Playertable[i].suit==3)
            {
                d++;
            }

        }
        if(a==3 or b==3 or c==3 or d==3)
        {
            return true;
        }
    }
    if(m==4){
        y=3;
        for(int i=0; i<y; i++)
        {
            if(Playertable[i].suit==0)
            {
                a++;
            }
            if(Playertable[i].suit==1)
            {
                b++;
            }
            if(Playertable[i].suit==2)
            {
                c++;
            }
            if(Playertable[i].suit==3)
            {
                d++;
            }

        }
        if(a==3 or b==3 or c==3 or d==3)
        {
            return true;
        }
    }
    if(m==5){
        y=4;
        for(int i=0; i<y; i++)
        {
            if(Playertable[i].suit==0)
            {
                a++;
            }
            if(Playertable[i].suit==1)
            {
                b++;
            }
            if(Playertable[i].suit==2)
            {
                c++;
            }
            if(Playertable[i].suit==3)
            {
                d++;
            }

        }
        if(a==3 or b==3 or c==3 or d==3)
        {
            return true;
        }
    }
    return false;
}
bool straightflush(Card Playertable[7], int m)
{

    int x,y;
    Card z;
    if (m==0)
    {
        y=2;
        return false;
    }
    if (m==1)
    {
        y=5;
        Card tab[y];
        for(int i=0; i<y; i++)
        {
            tab[i]=Playertable[i];
        }
        for (int i=0; i<y; i++)
        {
            for(int j=0; j<y-1; j++)
            {
                if(tab[j].rank>tab[j+1].rank)
                {
                    z=tab[j+1];
                    tab[j+1]=tab[j];
                    tab[j]=z;
                }
            }
        }
        if((tab[0].rank==tab[1].rank-1)and(tab[1].rank-1==tab[2].rank-2)and(tab[2].rank-2==tab[3].rank-3)and(tab[3].rank-3==tab[4].rank-4) and (tab[0].suit==tab[1].suit) and (tab[1].suit==tab[2].suit) and(tab[2].suit==tab[3].suit)and(tab[3].suit==tab[4].suit))
        {
            return true;
        }
    }
    if (m==2)
    {
        y=6;
        Card tab[y];
        for(int i=0; i<y; i++)
        {
            tab[i]=Playertable[i];
        }
        for (int i=0; i<y; i++)
        {
            for(int j=0; j<y-1; j++)
            {
                if(tab[j].rank>tab[j+1].rank)
                {
                    z=tab[j+1];
                    tab[j+1]=tab[j];
                    tab[j]=z;
                }
            }
        }
        if((tab[0].rank==tab[1].rank-1)and(tab[1].rank-1==tab[2].rank-2)and(tab[2].rank-2==tab[3].rank-3)and(tab[3].rank-3==tab[4].rank-4) and (tab[0].suit==tab[1].suit) and (tab[1].suit==tab[2].suit) and(tab[2].suit==tab[3].suit)and(tab[3].suit==tab[4].suit))
        {
            return true;
        }
        if((tab[5].rank-5==tab[1].rank-1)and(tab[1].rank-1==tab[2].rank-2)and(tab[2].rank-2==tab[3].rank-3)and(tab[3].rank-3==tab[4].rank-4) and (tab[5].suit==tab[1].suit) and (tab[1].suit==tab[2].suit) and(tab[2].suit==tab[3].suit)and(tab[3].suit==tab[4].suit))
        {
            return true;
        }
    }
    if (m==3)
    {
        y=7;
        Card tab[y];
        for(int i=0; i<y; i++)
        {
            tab[i]=Playertable[i];
        }
        for (int i=0; i<y; i++)
        {
            for(int j=0; j<y-1; j++)
            {
                if(tab[j].rank>tab[j+1].rank)
                {
                    z=tab[j+1];
                    tab[j+1]=tab[j];
                    tab[j]=z;
                }
            }
        }
        if((tab[0].rank==tab[1].rank-1)and(tab[1].rank-1==tab[2].rank-2)and(tab[2].rank-2==tab[3].rank-3)and(tab[3].rank-3==tab[4].rank-4) and (tab[0].suit==tab[1].suit) and (tab[1].suit==tab[2].suit) and(tab[2].suit==tab[3].suit)and(tab[3].suit==tab[4].suit))
        {
            return true;
        }
        if((tab[5].rank-5==tab[1].rank-1)and(tab[1].rank-1==tab[2].rank-2)and(tab[2].rank-2==tab[3].rank-3)and(tab[3].rank-3==tab[4].rank-4) and (tab[5].suit==tab[1].suit) and (tab[1].suit==tab[2].suit) and(tab[2].suit==tab[3].suit)and(tab[3].suit==tab[4].suit))
        {
            return true;
        }
        if((tab[6].rank-6==tab[5].rank-5)and(tab[6].rank-6==tab[2].rank-2)and(tab[2].rank-2==tab[3].rank-3)and(tab[3].rank-3==tab[4].rank-4) and (tab[5].suit==tab[6].suit) and (tab[6].suit==tab[2].suit) and(tab[2].suit==tab[3].suit)and(tab[3].suit==tab[4].suit))
        {
            return true;
        }
    }
    if(m==4){
        return false;
    }
    if(m==5){
        return false;
    }

    return false;

}
bool threetostraight(Card Playertable[7], int m)
{
    int x,y,z;
    if (m==0)
    {
        y=2;
        return false;
    }
    if (m==1)
    {
        y=5;
        int tab[5];
        for(int i=0; i<5; i++)
        {
            tab[i]=Playertable[i].rank;
        }
        for (int i=0; i<5; i++)
        {
            for(int j=0; j<4; j++)
            {
                if(tab[j]>tab[j+1])
                {
                    z=tab[j+1];
                    tab[j+1]=tab[j];
                    tab[j]=z;
                }
            }
        }
        if((tab[0]==tab[1]-1 and tab[1]-1==tab[2]-2) or (tab[1]==tab[2]-1 and tab[2]-1==tab[3]-2) or (tab[2]==tab[3]-1 and tab[3]-1==tab[4]-2))
        {
            return true;
        }
    }
    if (m==2)
    {
        y=6;
        int tab[6];
        for(int i=0; i<6; i++)
        {
            tab[i]=Playertable[i].rank;
        }
        for (int i=0; i<6; i++)
        {
            for(int j=0; j<5; j++)
            {
                if(tab[j]>tab[j+1])
                {
                    z=tab[j+1];
                    tab[j+1]=tab[j];
                    tab[j]=z;
                }
            }
        }
        if((tab[0]==tab[1]-1 and tab[1]-1==tab[2]-2) or (tab[1]==tab[2]-1 and tab[2]-1==tab[3]-2) or (tab[2]==tab[3]-1 and tab[3]-1==tab[4]-2) or (tab[3]==tab[4]-1 and tab[4]-1==tab[5]-2))
            {
            return true;
            }
    }
    if (m==3)
    {
        y=7;
        int tab[7];
        for(int i=0; i<7; i++)
        {
            tab[i]=Playertable[i].rank;
        }
        for (int i=0; i<7; i++)
        {
            for(int j=0; j<6; j++)
            {
                if(tab[j]>tab[j+1])
                {
                    z=tab[j+1];
                    tab[j+1]=tab[j];
                    tab[j]=z;
                }
            }
        }
        if((tab[0]==tab[1]-1 and tab[1]-1==tab[2]-2) or (tab[1]==tab[2]-1 and tab[2]-1==tab[3]-2) or (tab[2]==tab[3]-1 and tab[3]-1==tab[4]-2) or (tab[3]==tab[4]-1 and tab[4]-1==tab[5]-2) or (tab[4]==tab[5]-1 and tab[5]-1==tab[6]-2))
        {
            return true;
        }
    }
    if(m==4){
        y=3;
        int tab[3];
        for(int i=0; i<3; i++)
        {
            tab[i]=Playertable[i].rank;
        }
        for (int i=0; i<3; i++)
        {
            for(int j=0; j<2; j++)
            {
                if(tab[j]>tab[j+1])
                {
                    z=tab[j+1];
                    tab[j+1]=tab[j];
                    tab[j]=z;
                }
            }
        }
        if((tab[0]==tab[1]-1 and tab[1]-1==tab[2]-2))
        {
            return true;
        }
    }
    if(m==5){
        y=4;
        int tab[4];
        for(int i=0; i<4; i++)
        {
            tab[i]=Playertable[i].rank;
        }
        for (int i=0; i<4; i++)
        {
            for(int j=0; j<3; j++)
            {
                if(tab[j]>tab[j+1])
                {
                    z=tab[j+1];
                    tab[j+1]=tab[j];
                    tab[j]=z;
                }
            }
        }
        if((tab[0]==tab[1]-1 and tab[1]-1==tab[2]-2) or (tab[1]==tab[2]-1 and tab[2]-1==tab[3]-2))
        {
            return true;
        }
    }
    return false;

}
bool fourtostraight(Card Playertable[7], int m)
{
    int x,y,z;
    if (m==0)
    {
        y=2;
        return false;
    }
    if (m==1)
    {
        y=5;
        int tab[5];
        for(int i=0; i<5; i++)
        {
            tab[i]=Playertable[i].rank;
        }
        for (int i=0; i<5; i++)
        {
            for(int j=0; j<4; j++)
            {
                if(tab[j]>tab[j+1])
                {
                    z=tab[j+1];
                    tab[j+1]=tab[j];
                    tab[j]=z;
                }
            }
        }
        if((tab[0]==tab[1]-1 and tab[1]-1==tab[2]-2 and tab[2]-2==tab[3]-3) or (tab[1]==tab[2]-1 and tab[2]-1==tab[3]-2 and tab[3]-2==tab[4]-3))
        {
            return true;
        }
    }
    if (m==2)
    {
        y=6;
        int tab[6];
        for(int i=0; i<6; i++)
        {
            tab[i]=Playertable[i].rank;
        }
        for (int i=0; i<6; i++)
        {
            for(int j=0; j<5; j++)
            {
                if(tab[j]>tab[j+1])
                {
                    z=tab[j+1];
                    tab[j+1]=tab[j];
                    tab[j]=z;
                }
            }
        }
        if((tab[0]==tab[1]-1 and tab[1]-1==tab[2]-2 and tab[2]-2==tab[3]-3) or (tab[1]==tab[2]-1 and tab[2]-1==tab[3]-2 and tab[3]-2==tab[4]-3) or (tab[2]==tab[3]-1 and tab[3]-1==tab[4]-2 and tab[4]-2==tab[5]-3))
        {
            return true;
        }
    }
    if (m==3)
    {
        y=7;
        int tab[7];
        for(int i=0; i<7; i++)
        {
            tab[i]=Playertable[i].rank;
        }
        for (int i=0; i<7; i++)
        {
            for(int j=0; j<6; j++)
            {
                if(tab[j]>tab[j+1])
                {
                    z=tab[j+1];
                    tab[j+1]=tab[j];
                    tab[j]=z;
                }
            }
        }
        if((tab[0]==tab[1]-1 and tab[1]-1==tab[2]-2 and tab[2]-2==tab[3]-3) or (tab[1]==tab[2]-1 and tab[2]-1==tab[3]-2 and tab[3]-2==tab[4]-3) or (tab[2]==tab[3]-1 and tab[3]-1==tab[4]-2 and tab[4]-2==tab[5]-3) or (tab[3]==tab[4]-1 and tab[4]-1==tab[5]-2 and tab[5]-2==tab[6]-3))
        {
            return true;
        }
    }
    if(m==4){
        return false;
    }
    if(m==5){
        y=4;
        int tab[4];
        for(int i=0; i<4; i++)
        {
            tab[i]=Playertable[i].rank;
        }
        for (int i=0; i<4; i++)
        {
            for(int j=0; j<3; j++)
            {
                if(tab[j]>tab[j+1])
                {
                    z=tab[j+1];
                    tab[j+1]=tab[j];
                    tab[j]=z;
                }
            }
        }
        if(tab[0]==tab[1]-1 and tab[1]-1==tab[2]-2 and tab[2]-2==tab[3]-3)
        {
            return true;
        }
    }
    return false;

}
double chanceToWin(vector<Card>vec, int m, int l)  ///m-tura, l-liczba graczy
{
    int x,y;
    double chance=0;
    Card Player[2];
    Card Table[5];
    Card Playertable[7];
    for(int i=0;i<vec.size();i++){
        Playertable[i]=vec[i];
    }

    Player[0]=Playertable[0];
    Player[1]=Playertable[1];

    Table[0]=Playertable[2];
    Table[1]=Playertable[3];
    Table[2]=Playertable[4];
    Table[3]=Playertable[5];
    Table[4]=Playertable[6];

    Card Tableplayer[7];

    Tableplayer[0]=Table[0];
    Tableplayer[1]=Table[1];
    Tableplayer[2]=Table[2];
    Tableplayer[3]=Table[3];
    Tableplayer[4]=Table[4];
    Tableplayer[5]=Player[0];
    Tableplayer[6]=Player[1];

    if (m==0)
    {
        y=2;
        if(Playertable[0].rank==Playertable[1].rank)
        {
            if(Playertable[0].rank==12)
            {
                chance=(Playertable[0].rank+1)*(Playertable[0].rank+1)/6+15;
            }
            else
            {
                chance=(Playertable[0].rank+1)*(Playertable[0].rank+1)/6+12;
            }
        }
        else
        {
            if(Playertable[0].suit==Playertable[1].suit)
            {
                chance=chance+4;
            }
            chance=chance+(Playertable[0].rank+Playertable[1].rank)/2+1;
            if(Playertable[0].rank==12)
            {
                chance=chance+4;
            }
            if(Playertable[1].rank==12)
            {
                chance=chance+4;
            }
            if(Playertable[0].rank==11)
            {
                chance=chance+3;
            }
            if(Playertable[1].rank==11)
            {
                chance=chance+3;
            }
            if(Playertable[0].rank==10)
            {
                chance=chance+2;
            }
            if(Playertable[1].rank==10)
            {
                chance=chance+2;
            }
            if(Playertable[0].rank==9)
            {
                chance=chance+1;
            }
            if(Playertable[1].rank==9)
            {
                chance=chance+1;
            }
            if(Playertable[0].rank-Playertable[1].rank==4)
            {
                chance=chance+0.5;
            }
            if(Playertable[1].rank-Playertable[0].rank==4)
            {
                chance=chance+0.5;
            }
            if(Playertable[0].rank-Playertable[1].rank==3)
            {
                chance=chance+1.5;
            }
            if(Playertable[1].rank-Playertable[0].rank==3)
            {
                chance=chance+1.5;
            }
            if(Playertable[0].rank-Playertable[1].rank==2)
            {
                chance=chance+2.5;
            }
            if(Playertable[1].rank-Playertable[0].rank==2)
            {
                chance=chance+2.5;
            }
            if(Playertable[0].rank-Playertable[1].rank==1)
            {
                chance=chance+3.5;
            }
            if(Playertable[1].rank-Playertable[0].rank==1)
            {
                chance=chance+3.5;
            }
            chance=chance+1;
            //chance=chance/2; ///xd
        }
    }
    if (m==1)
    {
        y=5;
        if(straight(Playertable, 1))
        {
            chance=chance+37;
        }
        else
        {
            if(fourtostraight(Playertable, 1))
            {
                chance=chance+10;
            }
            else
            {
                if(threetostraight(Playertable, 1))
                {
                    if(!threetostraight(Tableplayer, 4)){
                        chance=chance+1;
                    }
                }
            }
        }
        if(flush(Playertable, 1))
        {
            chance=chance+60;
        }
        else
        {
            if(fourtoflush(Playertable, 1))
            {
                chance=chance+30;
            }
            else
            {
                if(threetoflush(Playertable, 1))
                {
                    if(!threetoflush(Tableplayer, 4)){
                    chance=chance+2;
                    }
                }
            }
        }
        if (twopair(Playertable,1))
        {
            chance=chance+36;
        }
        else
        {
            if(pairr(Playertable, 1))
            {
                if(!pairr(Tableplayer, 4)){
                chance=chance+15;
                }
            }
        }
        if(four(Playertable, 1))
        {
            chance=chance+79;
        }
        else
        {
            if(three(Playertable, 1))
            {
                if(!three(Tableplayer, 4)){
                chance=chance+54;
                }
            }
        }
        chance=chance+(Playertable[0].rank+Playertable[1].rank)/2;





    }
    if (m==2)
    {
        y=6;
        if(straight(Playertable, 2))
        {
            chance=chance+37;
        }
        else
        {
            if(fourtostraight(Playertable, 2))
            {
                if(!fourtostraight(Tableplayer, 5)){
                chance=chance+10;
                }
            }
            else
            {
                if(threetostraight(Playertable, 2))
                {
                    if(!threetostraight(Tableplayer, 5)){
                    chance=chance+1;
                    }
                }
            }
        }
        if(flush(Playertable, 2))
        {
            chance=chance+60;
        }
        else
        {
            if(fourtoflush(Playertable, 2))
            {
                if(!fourtoflush(Tableplayer, 5)){
                chance=chance+30;
                }
            }
            else
            {
                if(threetoflush(Playertable, 2))
                {
                    if(!threetoflush(Tableplayer, 5)){
                    chance=chance+2;
                    }
                }
            }
        }
        if (twopair(Playertable,2))
        {
            if(!twopair(Tableplayer, 5)){
            chance=chance+36;
            }
        }
        else
        {
            if(pairr(Playertable, 2))
            {
                if(!pairr(Tableplayer, 5)){
                chance=chance+14;
            }
            }
        }
        if(four(Playertable, 2))
        {
            if(!four(Tableplayer, 5)){
            chance=chance+79;
        }
        }
        else
        {
            if(three(Playertable, 2))
            {
                if(!three(Tableplayer, 5)){
                chance=chance+48;
            }
            }
        }
        chance=(chance+(Playertable[0].rank+Playertable[1].rank)/2)*9/10;
    }
    if (m==3)
    {
        y=7;
        if(straightflush(Playertable, 3))
        {
            if(!straightflush(Tableplayer, 1)){
            chance=chance+90;
            }
        }
        else
        {
            if(straight(Playertable, 3))
            {
                if(!straight(Tableplayer, 1)){
                chance=chance+19;
            }
            }
            if(flush(Playertable, 3))
            {
                if(!flush(Tableplayer, 1)){
                chance=chance+48;
            }
            }
        }
        if (full(Playertable,3))
        {
            if(!full(Tableplayer, 1)){
            chance=chance+66;
        }
        }
        if(four(Playertable, 3))
        {
            if(!four(Tableplayer, 1)){
            chance=chance+79;
        }
        }
        else
        {
            if(three(Playertable, 3))
            {
                if(!three(Tableplayer, 1)){
                chance=chance+36;
            }
            }
        }
        if(twopair(Playertable, 3)){
            if(!twopair(Tableplayer, 1)){
                chance=chance+25;
            }
        }
        if(pairr(Playertable, 3)){
            if(!pairr(Tableplayer, 1)){
                chance=chance+15;
            }
        }
        chance=chance+(Playertable[0].rank+Playertable[1].rank)/4;
    }

    chance=chance+(((100-chance)/6)*(7-l))/l;
    return chance;
}
void loadTextures()///and set textures
{
    texBackground.loadFromFile("grafika\\background.bmp");
    texCards.loadFromFile("grafika\\cards.png");
    texGameOver.loadFromFile("grafika\\gameover.png");
    texNewGame[0].loadFromFile("grafika\\newgame0.png");
    texNewGame[1].loadFromFile("grafika\\newgame1.png");
    player[1].tex0.loadFromFile("grafika\\kaczor v1.png");
    player[1].tex1.loadFromFile("grafika\\kaczor v2.png");
    player[1].tex2.loadFromFile("grafika\\kaczor v3.png");
    player[2].tex0.loadFromFile("grafika\\kim v1.png");
    player[2].tex1.loadFromFile("grafika\\kim v2.png");
    player[2].tex2.loadFromFile("grafika\\kim v3.png");
    player[3].tex0.loadFromFile("grafika\\kubis v1.png");
    player[3].tex1.loadFromFile("grafika\\kubis v2.png");
    player[3].tex2.loadFromFile("grafika\\kubis v3.png");
    player[4].tex0.loadFromFile("grafika\\scarlet v1.png");
    player[4].tex1.loadFromFile("grafika\\scarlet v2.png");
    player[4].tex2.loadFromFile("grafika\\scarlet v3.png");
    player[5].tex0.loadFromFile("grafika\\lysy v1.png");
    player[5].tex1.loadFromFile("grafika\\lysy v2.png");
    player[5].tex2.loadFromFile("grafika\\lysy v3.png");
    player[6].tex0.loadFromFile("grafika\\trump v1.png");
    player[6].tex1.loadFromFile("grafika\\trump v2.png");
    player[6].tex2.loadFromFile("grafika\\trump v3.png");
    texCall[0].loadFromFile("grafika\\callo.png");
    texCall[1].loadFromFile("grafika\\callw.png");
    texCall[2].loadFromFile("grafika\\callg.png");
    texPot[0].loadFromFile("grafika\\poto.png");
    texPot[1].loadFromFile("grafika\\potw.png");
    texPot[2].loadFromFile("grafika\\potg.png");
    texHalfPot[0].loadFromFile("grafika\\halfpoto.png");
    texHalfPot[1].loadFromFile("grafika\\halfpotw.png");
    texHalfPot[2].loadFromFile("grafika\\halfpotg.png");
    texAllIn[0].loadFromFile("grafika\\allinr.png");
    texAllIn[1].loadFromFile("grafika\\allinw.png");
    texAllIn[2].loadFromFile("grafika\\alling.png");
    texFold[0].loadFromFile("grafika\\foldo.png");
    texFold[1].loadFromFile("grafika\\foldw.png");
    texFold[2].loadFromFile("grafika\\foldg.png");
    texCheck[0].loadFromFile("grafika\\checko.png");
    texCheck[1].loadFromFile("grafika\\checkw.png");
    texCheck[2].loadFromFile("grafika\\checkg.png");
    texRaise[0].loadFromFile("grafika\\raiseo.png");
    texRaise[1].loadFromFile("grafika\\raisew.png");
    texRaise[2].loadFromFile("grafika\\raiseg.png");
    tex100[0].loadFromFile("grafika\\100o.png");
    tex100[1].loadFromFile("grafika\\100w.png");
    tex100[2].loadFromFile("grafika\\100g.png");
    tex200[0].loadFromFile("grafika\\200o.png");
    tex200[1].loadFromFile("grafika\\200w.png");
    tex200[2].loadFromFile("grafika\\200g.png");
    tex500[0].loadFromFile("grafika\\500o.png");
    tex500[1].loadFromFile("grafika\\500w.png");
    tex500[2].loadFromFile("grafika\\500g.png");
    tex1000[0].loadFromFile("grafika\\1000o.png");
    tex1000[1].loadFromFile("grafika\\1000w.png");
    tex1000[2].loadFromFile("grafika\\1000g.png");
    tex2000[0].loadFromFile("grafika\\2000o.png");
    tex2000[1].loadFromFile("grafika\\2000w.png");
    tex2000[2].loadFromFile("grafika\\2000g.png");
    texNumbers[0].loadFromFile("grafika\\0.png");
    texNumbers[1].loadFromFile("grafika\\1.png");
    texNumbers[2].loadFromFile("grafika\\2.png");
    texNumbers[3].loadFromFile("grafika\\3.png");
    texNumbers[4].loadFromFile("grafika\\4.png");
    texNumbers[5].loadFromFile("grafika\\5.png");
    texNumbers[6].loadFromFile("grafika\\6.png");
    texNumbers[7].loadFromFile("grafika\\7.png");
    texNumbers[8].loadFromFile("grafika\\8.png");
    texNumbers[9].loadFromFile("grafika\\9.png");
    texNumbers[10].loadFromFile("grafika\\null.png");
    texMoneyIcon.loadFromFile("grafika\\moneyicon.png");
    for(int i=0;i<4;i++)
    {
        for(int j=0;j<13;j++)
        {

            sprCards[i*13+j].setTexture(texCards);
            sprCards[i*13+j].setTextureRect(IntRect(79*j,123*i,79,123));
        }
    }
    sprBackground.setTexture(texBackground);
    sprCheckOrCall.setTexture(texCheck[2]);
    sprRaise.setTexture(texRaise[2]);
    sprFold.setTexture(texFold[2]);
    spr100.setTexture(tex100[2]);
    spr200.setTexture(tex200[2]);
    spr500.setTexture(tex500[2]);
    spr1000.setTexture(tex1000[2]);
    spr2000.setTexture(tex2000[2]);
    sprPot.setTexture(texPot[2]);
    sprHalfPot.setTexture(texHalfPot[2]);
    sprAllIn.setTexture(texAllIn[2]);
    sprGameOver.setTexture(texGameOver);
    sprNewGame.setTexture(texNewGame[0]);
}
void setMyTexturesDeafult()
{
    sprCheckOrCall.setTexture(texCheck[2]);
    sprRaise.setTexture(texRaise[2]);
    sprFold.setTexture(texFold[2]);
    spr100.setTexture(tex100[2]);
    spr200.setTexture(tex200[2]);
    spr500.setTexture(tex500[2]);
    spr1000.setTexture(tex1000[2]);
    spr2000.setTexture(tex2000[2]);
    sprPot.setTexture(texPot[2]);
    sprHalfPot.setTexture(texHalfPot[2]);
    sprAllIn.setTexture(texAllIn[2]);
    drawRaiseButtons=0;
}
void loadCards()
{
    emptyCard.rank=-1;
    emptyCard.suit=-1;
   // ifstream file;
    //file.open("cards.txt");
    /*int i=0;
    while(!file.eof())
    {
        file>>deck[i].ID;
        file>>deck[i].rank;
        file>>deck[i].suit;
        deck[i].sprCard=sprCards[i];
        i++;
    }*/
    for(int i=0;i<4;i++)
    {
        for(int j=0;j<13;j++)
        {
            deck[i*13+j].ID=i*13+j;
            deck[i*13+j].rank=j;
            deck[i*13+j].suit=i;
            deck[i*13+j].sprCard=sprCards[i*13+j];
        }
    }
   // file.close();
}
bool sameRanks(const Card c1, const Card c2)
{
    if(c1.rank==c2.rank)
        return true;
    return false;
}
bool sameSuits(const Card c1,const  Card c2)
{
    if(c1.suit==c2.suit)
        return true;
    return false;
}
void declarePlayers()
{
    ifstream file;
    file.open("players.txt");
    for(int i=0;i<7;i++)
    {
        file>>player[i].ID;
        file>>player[i].totalMoney;
        player[i].moneyIn=0;
        player[i].nr=i;
        player[i].inRound=true;
        player[i].inGame=true;
    }
    file.close();
    for(int i=1;i<7;i++)
        player[i].sprAvatar.setTexture(player[i].tex1);
}
void drawing()
{
    int los,i=0,j;
    int wylosowane[19];
    while(i<19)
    {
        los=rand()%52;
        j=0;
        while(j<i)
        {
            if(los==wylosowane[j])
                break;
            j++;
        }
        if(i==j)
            wylosowane[i]=los,i++;

    }

    for(int i=0;i<7;i++)
    {
        player[i].card1=deck[wylosowane[2*i]];
        player[i].card2=deck[wylosowane[2*i+1]];
    }
    for(int i=0;i<5;i++)
        cardsOnTable[i]=deck[wylosowane[i+14]];
}
void setSpritePositions()
{
    cardsOnTable[0].sprCard.setPosition(380,324);
    cardsOnTable[1].sprCard.setPosition(470,324);
    cardsOnTable[2].sprCard.setPosition(560,324);
    cardsOnTable[3].sprCard.setPosition(650,324);
    cardsOnTable[4].sprCard.setPosition(740,324);
    player[0].card1.sprCard.setPosition(520,560);
    player[0].card2.sprCard.setPosition(600,560);
    player[1].card1.sprCard.setPosition(320,500);
    player[1].card2.sprCard.setPosition(350,540);
    player[2].card1.sprCard.setPosition(200,320);
    player[2].card2.sprCard.setPosition(230,360);
    player[3].card1.sprCard.setPosition(315,140);
    player[3].card2.sprCard.setPosition(350,175);
    player[4].card1.sprCard.setPosition(780,130);
    player[4].card2.sprCard.setPosition(805,165);
    player[5].card1.sprCard.setPosition(910,320);
    player[5].card2.sprCard.setPosition(940,360);
    player[6].card1.sprCard.setPosition(780,500);
    player[6].card2.sprCard.setPosition(810,540);
    player[1].sprAvatar.setPosition(190,560);
    player[2].sprAvatar.setPosition(65,325);
    player[3].sprAvatar.setPosition(200,50);
    player[4].sprAvatar.setPosition(915,50);
    player[5].sprAvatar.setPosition(1035,325);
    player[6].sprAvatar.setPosition(915,560);
    sprFold.setPosition(260,735);
    sprCheckOrCall.setPosition(494,735);
    sprRaise.setPosition(728,735);
    sprHalfPot.setPosition(955,760);
    sprPot.setPosition(955,722);
    sprAllIn.setPosition(955,680);
    spr100.setPosition(1100,761);
    spr200.setPosition(1100,727);
    spr500.setPosition(1100,693);
    spr1000.setPosition(1100,659);
    spr2000.setPosition(1100,625);
    player[0].sprTotalMoney[0].setPosition(540,693);
    player[0].sprMoneyIn[0].setPosition(540,515);
    player[1].sprTotalMoney[0].setPosition(185,680);
    player[1].sprMoneyIn[0].setPosition(300,500);
    player[2].sprTotalMoney[0].setPosition(55,450);
    player[2].sprMoneyIn[0].setPosition(235,380);
    player[3].sprTotalMoney[0].setPosition(185,170);
    player[3].sprMoneyIn[0].setPosition(290,250);
    player[4].sprTotalMoney[0].setPosition(905,170);
    player[4].sprMoneyIn[0].setPosition(800,250);
    player[5].sprTotalMoney[0].setPosition(1025,450);
    player[5].sprMoneyIn[0].setPosition(870,380);
    player[6].sprTotalMoney[0].setPosition(905,680);
    player[6].sprMoneyIn[0].setPosition(775,500);
    sprMoneyOnTable[0].setPosition(535,260);
    for(int i=1;i<5;i++)
    {
        player[0].sprTotalMoney[i].setPosition(555+20*i,693);
        player[0].sprMoneyIn[i].setPosition(555+20*i,515);
        player[1].sprTotalMoney[i].setPosition(200+20*i,680);
        player[1].sprMoneyIn[i].setPosition(305+20*i,500);
        player[2].sprTotalMoney[i].setPosition(70+20*i,450);
        player[2].sprMoneyIn[i].setPosition(250+20*i,380);
        player[3].sprTotalMoney[i].setPosition(200+20*i,170);
        player[3].sprMoneyIn[i].setPosition(305+20*i,250);
        player[4].sprTotalMoney[i].setPosition(920+20*i,170);
        player[4].sprMoneyIn[i].setPosition(815+20*i,250);
        player[5].sprTotalMoney[i].setPosition(1040+20*i,450);
        player[5].sprMoneyIn[i].setPosition(885+20*i,380);
        player[6].sprTotalMoney[i].setPosition(920+20*i,680);
        player[6].sprMoneyIn[i].setPosition(790+20*i,500);
        sprMoneyOnTable[i].setPosition(550+20*i,260);
    }
    sprNewGame.setPosition(510,570);

}
int allInPlayers()
{
    int sum=0;
    for(int i=0;i<7;i++)
    {
        if(player[i].isAllIn==true)
            sum++;
    }
    return sum;
}
void setMoneyTexture(Sprite spr[5], int money)
{
    if(money==0)
        spr[0].setTexture(texNumbers[10]);
    else
        spr[0].setTexture(texMoneyIcon);
    for(int i=4;i>0;i--)
    {
        if(money>0)
            spr[i].setTexture(texNumbers[money%10]);
        else
            spr[i].setTexture(texNumbers[10]);
        money=money/10;
    }
}
void setAllMoneyTextures(const int moneyOnTable)
{
    for(int i=0;i<7;i++)
    {
        setMoneyTexture(player[i].sprMoneyIn,player[i].moneyIn);
        setMoneyTexture(player[i].sprTotalMoney,player[i].totalMoney);
    }
    setMoneyTexture(sprMoneyOnTable,moneyOnTable);
}
int playersInGame()
{
    int sum=0;
    for(int i=0;i<7;i++)
    {
        if(player[i].inGame==true)
            sum++;
    }
    return sum;
}
int playersInRound()
{
    int sum=0;
    for(int i=0;i<7;i++)
    {
        if(player[i].inRound==true)
            sum++;
    }
    return sum;
}
void setPlayersInRound()    ///and their textures
{

    for(int i=0;i<7;i++)
    {
        player[i].sumOfAllIn=player[i].totalMoney;
        player[i].moneyReceived=0;
        if(player[i].inGame==true)
        {
            player[i].sprAvatar.setTexture(player[i].tex1);
            player[i].inRound=true;
        }
        else
        {
            player[i].sprAvatar.setTexture(player[i].tex2);
            player[i].inRound=false;
        }

        player[i].isAllIn=false;
    }
}
vector<Card> fillCardsTo7(Card card1, Card card2,int whichStage)
{
    vector<Card> vec;
    vec.push_back(card1);
    vec.push_back(card2);
    whichStage--;
    if(whichStage>=0)
    {
        for(int i=0;i<3;i++)
            vec.push_back(cardsOnTable[i]);
    }
    whichStage--;
    if(whichStage>=0)
        vec.push_back(cardsOnTable[3]);

    whichStage--;
    if(whichStage>=0)
        vec.push_back(cardsOnTable[4]);

    for(int i=vec.size();i<7;i++)
        vec.push_back(emptyCard);
    return vec;
}
void drawSprites(RenderWindow &window, int whichStage,bool drawRaiseButtons)
{
    window.clear(Color::White);
    window.draw(sprBackground);
    window.draw(player[0].card1.sprCard);
    window.draw(player[0].card2.sprCard);
    window.draw(sprFold);
    window.draw(sprCheckOrCall);
    window.draw(sprRaise);
    for(int i=0;i<7;i++)
    {
        for(int j=0;j<5;j++)
        {
            window.draw(player[i].sprMoneyIn[j]);
            window.draw(player[i].sprTotalMoney[j]);
        }
    }
    for(int i=0;i<5;i++)
        window.draw(sprMoneyOnTable[i]);
    if(drawRaiseButtons)
    {
        window.draw(spr100);
        window.draw(spr200);
        window.draw(spr500);
        window.draw(spr1000);
        window.draw(spr2000);
        window.draw(sprHalfPot);
        window.draw(sprPot);
        window.draw(sprAllIn);
    }
    for(int i=1;i<7;i++)
        window.draw(player[i].sprAvatar);
    whichStage--;

    if(whichStage<0)
        return;
    for(int i=0;i<3;i++)
        window.draw(cardsOnTable[i].sprCard);
    whichStage--;
    if(whichStage<0)
        return;
    window.draw(cardsOnTable[3].sprCard);
    whichStage--;
    if(whichStage<0)
        return;
    window.draw(cardsOnTable[4].sprCard);
    whichStage--;
    if(whichStage<0)
        return;
    for(int i=1;i<7;i++)
    {
        if(player[i].inRound==true)
        {
            window.draw(player[i].card1.sprCard);
            window.draw(player[i].card2.sprCard);
        }
    }
}
Card isThereAPair(const Card example , const vector<Card> c)     /// czy jest para nie taka jak podana w argumencie
{
    for(int i=0;i<5;i++)
    {
        for(int j=i+1;j<5;j++)
        {
            if(sameRanks(c[i],c[j])&&!sameRanks(c[j],example))
                return c[i];
        }
    }

    return emptyCard;
}
Card isThereAThree( const vector<Card> c)     ///czy jest trojka
{
    for(int i=0;i<5;i++)
    {
        for(int j=i+1;j<5;j++)
        {
            for(int k=j+1;k<5;k++)
            {
                if(sameRanks(c[i],c[j])&&sameRanks(c[j],c[k]))
                return c[i];
            }
        }
    }

    return emptyCard;
}
Card isThereAFour( const vector<Card> c)    ///czy jest kareta
{
    for(int i=0;i<5;i++)
    {
        for(int j=i+1;j<5;j++)
        {
            for(int k=j+1;k<5;k++)
            {
                for(int n=k+1;n<5;n++)
                {
                    if(sameRanks(c[i],c[j])&&sameRanks(c[j],c[k])&&sameRanks(c[n],c[k]))
                    return c[i];
                }
            }
        }
    }
    return emptyCard;
}
Card areThereTwoPairs( const vector<Card> c) ///zwraca karte z wyzszej pary
{
    Card temp1;
    Card temp2;
    temp1=isThereAPair(emptyCard,c);
    if(temp1!=emptyCard)
    {
        temp2=isThereAPair(temp1,c);
        if(temp2!=emptyCard)
        {
            if(temp1.rank>temp2.rank)
                return temp1;
            return temp2;
        }
    }
    return emptyCard;
}
Card isThereAFullHouse(const vector<Card> c)         ///jesli jest full daje karte z trojki
{
    Card temp=isThereAThree(c);
    if(temp!=emptyCard)
    {
        if(isThereAPair(temp,c)!=emptyCard)
            return temp;
    }
    return emptyCard;
}
Card highestCard(Card ex1,Card ex2, const vector<Card> c) ///podaje najwyzsza karte nie podana w argumentach
{
    Card temp=c[0];
    for(int i=1;i<c.size();i++)
    {
        if(temp.rank<c[i].rank&&c[i]!=ex1&&c[i]!=ex2)
            temp=c[i];
    }
    return temp;
}
Card highestCard(Card ex1,Card ex2,Card ex3, const vector<Card> c) ///podaje najwyzsza karte nie podana w argumentach
{
    Card temp=c[0];
    for(int i=1;i<c.size();i++)
    {
        if(temp.rank<c[i].rank&&c[i]!=ex1&&c[i]!=ex2&&c[i]!=ex3)
            temp=c[i];
    }
    return temp;
}
Card highestCard(Card ex1,Card ex2,Card ex3,Card ex4, const vector<Card> c) ///podaje najwyzsza karte nie podana w argumentach
{
    Card temp=c[0];
    for(int i=1;i<c.size();i++)
    {
        if(temp.rank<c[i].rank&&c[i]!=ex1&&c[i]!=ex2&&c[i]!=ex3&&c[i]!=ex4)
            temp=c[i];
    }
    return temp;
}
Card isThereAFlush(const vector<Card> c) ///jesli jest kolor daje najwysza karte
{
    for(int i=1;i<5;i++)
    {
        if(sameSuits(c[0],c[i])==0)
            return emptyCard;
    }
    return highestCard(emptyCard,emptyCard,c);
}
Card isThereAStraight( vector<Card> c)
{
    sort(c.begin(),c.end());
    for(int i=0;i<4;i++)
    {
            if((c[i].rank+1)!=c[i+1].rank)
                return emptyCard;
    }
    return c[4];
}
Card isThereAPoker(const vector<Card> c)
{
    if(isThereAFlush(c)!=emptyCard &&isThereAStraight(c)!=emptyCard)
        return highestCard(emptyCard,emptyCard,c);
    return emptyCard;
}
bool operator< (const vector<Card> c1,const vector<Card> c2)
{
    if(isThereAPoker(c1)<isThereAPoker(c2)&&isThereAPoker(c2)!=emptyCard)
        return true;
    if(isThereAPoker(c2)<isThereAPoker(c1)&&isThereAPoker(c1)!=emptyCard)
        return false;
    if(isThereAFour(c1)<=isThereAFour(c2)&&isThereAFour(c2)!=emptyCard)
    {
        if(isThereAFour(c1)<isThereAFour(c2))
            return true;
        if(highestCard(emptyCard,isThereAFour(c1),c1)<highestCard(emptyCard,isThereAFour(c2),c2))
            return true;
        return false;
    }
    if(isThereAFour(c2)<=isThereAFour(c1)&&isThereAFour(c1)!=emptyCard)
    {
        if(isThereAFour(c2)<isThereAFour(c1))
            return false;
        if(highestCard(emptyCard,isThereAFour(c2),c2)<highestCard(emptyCard,isThereAFour(c1),c1))
            return false;
        return false;
    }
    if(isThereAFullHouse(c1)<=isThereAFullHouse(c2)&&isThereAFullHouse(c2)!=emptyCard)
    {
        if(isThereAFullHouse(c1)<isThereAFullHouse(c2))
            return true;
        if(highestCard(emptyCard,isThereAFullHouse(c1),c1)<highestCard(emptyCard,isThereAFullHouse(c2),c2))
            return true;
        return false;
    }
    if(isThereAFullHouse(c2)<=isThereAFullHouse(c1)&&isThereAFullHouse(c1)!=emptyCard)
    {
        if(isThereAFullHouse(c2)<isThereAFullHouse(c1))
            return false;
        if(highestCard(emptyCard,isThereAFullHouse(c2),c2)<highestCard(emptyCard,isThereAFullHouse(c1),c1))
            return false;
        return false;
    }
    if(isThereAFlush(c1)<=isThereAFlush(c2)&&isThereAFlush(c2)!=emptyCard)
    {
        if(isThereAFlush(c1)<isThereAFlush(c2))
            return true;
        return false;
    }
    if(isThereAFlush(c2)<=isThereAFlush(c1)&&isThereAFlush(c1)!=emptyCard)
    {
        if(isThereAFlush(c2)<isThereAFlush(c1))
            return false;
        return false;
    }
    if(isThereAStraight(c1)<=isThereAStraight(c2)&&isThereAStraight(c2)!=emptyCard)
    {
        if(isThereAStraight(c1)<isThereAStraight(c2))
            return true;
        return false;
    }
    if(isThereAStraight(c2)<=isThereAStraight(c1)&&isThereAStraight(c1)!=emptyCard)
    {
        if(isThereAStraight(c2)<isThereAStraight(c1))
            return false;
        return false;
    }
    if(isThereAThree(c1)<=isThereAThree(c2)&&isThereAThree(c2)!=emptyCard)
    {
        if(isThereAThree(c1)<isThereAThree(c2))
            return true;
        if(highestCard(emptyCard,isThereAThree(c1),c1)<=highestCard(emptyCard,isThereAThree(c2),c2))
        {
            Card t1=highestCard(emptyCard,isThereAThree(c1),c1);
            Card t2=highestCard(emptyCard,isThereAThree(c2),c2);
            if(t1<t2)
                return true;
            if(highestCard(t1,isThereAThree(c1),c1)<(highestCard(t2,isThereAThree(c2),c2)))
                return true;
        }
        return false;
    }
    if(isThereAThree(c2)<=isThereAThree(c1)&&isThereAThree(c1)!=emptyCard)
    {
        if(isThereAThree(c2)<isThereAThree(c1))
            return false;
        if(highestCard(emptyCard,isThereAThree(c2),c2)<=highestCard(emptyCard,isThereAThree(c1),c1))
        {
            Card t1=highestCard(emptyCard,isThereAThree(c2),c2);
            Card t2=highestCard(emptyCard,isThereAThree(c1),c1);
            if(t1<t2)
                return false;
            if(highestCard(t2,isThereAThree(c2),c2)<(highestCard(t1,isThereAThree(c1),c1)))
                return false;
        }
        return false;
    }
    if(areThereTwoPairs(c1)<=areThereTwoPairs(c2)&&areThereTwoPairs(c2)!=emptyCard)
    {
        if(areThereTwoPairs(c1)<areThereTwoPairs(c2))
            return false;
        if(isThereAPair(areThereTwoPairs(c1),c1)<=isThereAPair(areThereTwoPairs(c2),c2))
        {
            if(isThereAPair(areThereTwoPairs(c1),c1)<isThereAPair(areThereTwoPairs(c2),c2))
                return true;
            if(highestCard(areThereTwoPairs(c1),isThereAPair(areThereTwoPairs(c1),c1),c1)<highestCard(areThereTwoPairs(c2),isThereAPair(areThereTwoPairs(c2),c2),c2))
                return true;
        }
        return false;
    }
    if(areThereTwoPairs(c2)<=areThereTwoPairs(c1)&&areThereTwoPairs(c1)!=emptyCard)
    {
        if(areThereTwoPairs(c2)<areThereTwoPairs(c1))
            return false;
        if(isThereAPair(areThereTwoPairs(c2),c2)<=isThereAPair(areThereTwoPairs(c1),c1))
        {
            if(isThereAPair(areThereTwoPairs(c2),c2)<isThereAPair(areThereTwoPairs(c1),c1))
                return false;
            if(highestCard(areThereTwoPairs(c2),isThereAPair(areThereTwoPairs(c2),c2),c2)<highestCard(areThereTwoPairs(c1),isThereAPair(areThereTwoPairs(c1),c1),c1))
                return false;
        }
        return false;
    }
    if(isThereAPair(emptyCard,c1)<=isThereAPair(emptyCard,c2)&& isThereAPair(emptyCard,c2)!=emptyCard)
    {
        if(isThereAPair(emptyCard,c1)<isThereAPair(emptyCard,c2))
            return true;
        if(highestCard(isThereAPair(emptyCard,c1),emptyCard,c1)<=highestCard(isThereAPair(emptyCard,c2),emptyCard,c2))
        {
            if(highestCard(isThereAPair(emptyCard,c1),emptyCard,c1)<highestCard(isThereAPair(emptyCard,c2),emptyCard,c2))
                return true;

            if(highestCard(isThereAPair(emptyCard,c1),highestCard(isThereAPair(emptyCard,c1),emptyCard,c1),c1)<highestCard(isThereAPair(emptyCard,c2),highestCard(isThereAPair(emptyCard,c2),emptyCard,c2),c2))
                return true;
        }
        return false;
    }
    if(isThereAPair(emptyCard,c2)<=isThereAPair(emptyCard,c1)&& isThereAPair(emptyCard,c1)!=emptyCard)
    {
        if(isThereAPair(emptyCard,c2)<isThereAPair(emptyCard,c1))
            return false;
        if(highestCard(isThereAPair(emptyCard,c2),emptyCard,c2)<=highestCard(isThereAPair(emptyCard,c1),emptyCard,c1))
        {
            if(highestCard(isThereAPair(emptyCard,c2),emptyCard,c2)<highestCard(isThereAPair(emptyCard,c1),emptyCard,c1))
                return false;

            if(highestCard(isThereAPair(emptyCard,c2),highestCard(isThereAPair(emptyCard,c2),emptyCard,c2),c2)<highestCard(isThereAPair(emptyCard,c1),highestCard(isThereAPair(emptyCard,c1),emptyCard,c1),c1))
                return false;
        }
        return false;
    }
    vector<Card> temp1=c1,temp2=c2;
    sort(temp1.begin(),temp1.end());
    sort(temp2.begin(),temp2.end());
    for(int i=0;i<5;i++)
    {
        if(temp1[i]<temp2[i])
            return true;
        if(temp1[i]>temp2[i])
            return false;
    }

    return false;
}
void sortCardSets(vector<Card> vec[21])
{
    vector<Card> temp;
    for(int i=20;i>=0;i--)
    {
        for(int j=0;j<i;j++)
        {
            if(vec[j+1]<vec[j])
            {
                temp=vec[j];
                vec[j]=vec[j+1];
                vec[j+1]=temp;
            }
        }
    }
}
vector<Card> highestCardSet(const vector<Card> c,const bool inRound)
{
    if(!inRound)
    {
        vector<Card> vec;
        vec.push_back(deck[0]);
        vec.push_back(deck[14]);
        vec.push_back(deck[28]);
        vec.push_back(deck[42]);
        vec.push_back(deck[5]);
        return vec;
    }
    vector<Card> vex[21];
    int sum=-1;
    for(int i=0;i<7;i++)
    {
        for(int j=0;j<7;j++)
        {
            if(i<j)
            {
                sum++;
                for(int k=0;k<7;k++)
                {

                    if(k!=i&&k!=j)
                        vex[sum].push_back(c[k]);
                }
            }
        }
    }
    //sort(&vex[0],&vex[20]);
    sortCardSets(vex);
    return vex[20];
}
vector<int> nameTheWinner(const vector<Card> vec[7])        ///zwraca wektor punktow graczy odpowiadajacym kolejnosci najlepszych ulozen kart
{
    vector<int> points(7);
    for(int i=0;i<7;i++)
    {
        for(int j=0;j<7;j++)
        {
            if(vec[i]<vec[j])
                points[j]++;
        }
    }
    for(int i=0;i<7;i++)
    {
        if(player[i].inRound==false)
            points[i]=0;
    }
    return points;
}
void dealMoneyToWinners(int &moneyOntable)  ///moze sie sypac
{
    vector<Card> tem[7];
    for(int i=0;i<7;i++)
    {
        tem[i]=highestCardSet(fillCardsTo7(player[i].card1,player[i].card2,3),player[i].inRound);
    }
    vector<int> playerPoints=nameTheWinner(tem);
    int mostPoints=6;
    int howManyToDealMoney;
    int temp;
    int normalWage;
    while(moneyOntable>0)
    {
        howManyToDealMoney=0;
        for(int i=0;i<7;i++)
        {
            if(playerPoints[i]==mostPoints)
                howManyToDealMoney++;
        }
        while(moneyOntable>0&&howManyToDealMoney>0)
        {
            normalWage=moneyOntable/howManyToDealMoney;
            for(int i=0;i<7;i++)
            {
                if(playerPoints[i]==mostPoints)
                {
                    if(player[i].isAllIn)
                    {
                        if(player[i].sumOfAllIn*playersInRound()-player[i].moneyReceived>=normalWage)   ///jesli moze dostac wiecej niz jest do zgarniecia
                        {
                            temp=normalWage;
                            moneyOntable-=temp;
                            player[i].totalMoney+=temp;
                            player[i].moneyReceived+=temp;
                        }
                        else
                        {
                            temp=player[i].sumOfAllIn*playersInRound()/howManyToDealMoney;
                            moneyOntable-=temp;
                            player[i].totalMoney+=temp;
                            player[i].moneyReceived+=temp;
                            howManyToDealMoney--;
                        }
                    }
                    else
                    {
                        temp=normalWage;
                        moneyOntable-=temp;
                        player[i].totalMoney+=temp;
                        player[i].moneyReceived+=temp;
                    }
                }
            }
        }
        mostPoints--;
    }
}
void eventService(RenderWindow &window, const int time, const int whichStage,const int whosTurn,int &controller,const int pot,const int smallBlind, const int bigBlind)
{
    Clock clock;
    Event event;
    while(clock.getElapsedTime().asSeconds()<time)
    {
        int x,y;
        do
        {

            if(event.type== Event::Closed)
                window.close(),exit(0);
            if(whosTurn==0)
            {
                x=Mouse::getPosition(window).x;
                y=Mouse::getPosition(window).y;
                if(x>=260&& x<=472&&y>=735&&y<=779) ///myszka na fold
                {
                    sprFold.setTexture(texFold[0]);
                    drawSprites(window,whichStage,drawRaiseButtons);
                    window.display();
                    if(event.type==Event::MouseButtonPressed)
                    {
                        controller=1;
                        return;
                    }
                }
                else
                {
                    sprFold.setTexture(texFold[1]);
                    drawSprites(window,whichStage,drawRaiseButtons);
                    window.display();
                }

                if(x>=494&& x<=707&&y>=735&&y<=779)     ///myszka na check/call
                {
                    if(player[0].moneyIn==pot)
                    {
                        sprCheckOrCall.setTexture(texCheck[0]);
                    }
                    else
                    {
                        if(canIGo(pot-player[0].moneyIn))
                            sprCheckOrCall.setTexture(texCall[0]);
                        else
                            sprCheckOrCall.setTexture(texCall[2]);
                    }
                    drawSprites(window,whichStage,drawRaiseButtons);
                    window.display();
                    if(event.type==Event::MouseButtonPressed&&canIGo(pot-player[0].moneyIn))
                    {
                        controller=2;
                        return;
                    }
                }
                else
                {
                    if(player[0].moneyIn==pot)
                        sprCheckOrCall.setTexture(texCheck[1]);
                    else
                        sprCheckOrCall.setTexture(texCall[1]);
                    drawSprites(window,whichStage,drawRaiseButtons);
                    window.display();
                }

                if(x>=728&& x<=940&&y>=735&&y<=779)     ///myszka na raise
                {
                    sprRaise.setTexture(texRaise[0]);
                    drawSprites(window,whichStage,drawRaiseButtons);
                    window.display();
                    if(event.type==Event::MouseButtonPressed)
                        drawRaiseButtons=1;
                }
                else
                {
                    sprRaise.setTexture(texRaise[1]);
                    drawSprites(window,whichStage,drawRaiseButtons);
                    window.display();
                }
                if(canIGo(pot-player[0].moneyIn+smallBlind)&&drawRaiseButtons==1)
                {
                    if(x>=955&& x<=1093&&y>=760&&y<=795)    ///myszka na half pot
                    {
                        sprHalfPot.setTexture(texHalfPot[0]);
                        drawSprites(window,whichStage,drawRaiseButtons);
                        window.display();
                        if(event.type==Event::MouseButtonPressed)
                        {
                            controller=smallBlind;
                            return;
                        }

                    }
                    else
                    {
                        sprHalfPot.setTexture(texHalfPot[1]);
                        drawSprites(window,whichStage,drawRaiseButtons);
                        window.display();
                    }
                }
                else
                {
                    sprHalfPot.setTexture(texHalfPot[2]);
                    drawSprites(window,whichStage,drawRaiseButtons);
                    window.display();
                }
                if(canIGo(pot-player[0].moneyIn+bigBlind)&&drawRaiseButtons==1)
                {
                    if(x>=955&& x<=1093&&y>=722&&y<=757)       ///myszka na pot
                    {
                        sprPot.setTexture(texPot[0]);
                        drawSprites(window,whichStage,drawRaiseButtons);
                        window.display();
                        if(event.type==Event::MouseButtonPressed)
                        {
                            controller=bigBlind;
                            return;
                        }
                    }
                    else
                    {
                        sprPot.setTexture(texPot[1]);
                        drawSprites(window,whichStage,drawRaiseButtons);
                        window.display();
                    }
                }
                else
                {
                    sprPot.setTexture(texPot[2]);
                    drawSprites(window,whichStage,drawRaiseButtons);
                    window.display();
                }

                if(x>=955&& x<=1093&&y>=680&&y<=715)        ///myszka na all in
                {
                    sprAllIn.setTexture(texAllIn[0]);
                    drawSprites(window,whichStage,drawRaiseButtons);
                    window.display();
                    if(event.type==Event::MouseButtonPressed)
                    {
                        controller=-1;
                        return;
                    }
                }
                else
                {
                    sprAllIn.setTexture(texAllIn[1]);
                    drawSprites(window,whichStage,drawRaiseButtons);
                    window.display();
                }
                if(canIGo(100)&&drawRaiseButtons==1)
                {
                    if(x>=1100&& x<=1185&&y>=761&&y<=795)       ///100
                    {
                        spr100.setTexture(tex100[0]);
                        drawSprites(window,whichStage,drawRaiseButtons);
                        window.display();
                        if(event.type==Event::MouseButtonPressed)
                        {
                            controller=100;
                            return;
                        }
                    }
                    else
                    {
                        spr100.setTexture(tex100[1]);
                        drawSprites(window,whichStage,drawRaiseButtons);
                        window.display();
                    }
                }
                if(canIGo(200)&&drawRaiseButtons==1)
                {
                    if(x>=1100&& x<=1185&&y>=727&&y<=761)       ///200
                    {
                        spr200.setTexture(tex200[0]);
                        drawSprites(window,whichStage,drawRaiseButtons);
                        window.display();
                        if(event.type==Event::MouseButtonPressed)
                        {
                            controller=200;
                            return;
                        }
                    }
                    else
                    {
                        spr200.setTexture(tex200[1]);
                        drawSprites(window,whichStage,drawRaiseButtons);
                        window.display();
                    }
                }
                if(canIGo(500)&&drawRaiseButtons==1)
                {
                    if(x>=1100&& x<=1185&&y>=693&&y<=727)       ///500
                    {
                        spr500.setTexture(tex500[0]);
                        drawSprites(window,whichStage,drawRaiseButtons);
                        window.display();
                        if(event.type==Event::MouseButtonPressed)
                        {
                            controller=500;
                            return;
                        }
                    }
                    else
                    {
                        spr500.setTexture(tex500[1]);
                        drawSprites(window,whichStage,drawRaiseButtons);
                        window.display();
                    }
                }
                if(canIGo(1000)&&drawRaiseButtons==1)
                {
                    if(x>=1100&& x<=1185&&y>=659&&y<=693)       ///1000
                    {
                        spr1000.setTexture(tex1000[0]);
                        drawSprites(window,whichStage,drawRaiseButtons);
                        window.display();
                        if(event.type==Event::MouseButtonPressed)
                        {
                            controller=1000;
                            return;
                        }
                    }
                    else
                    {
                        spr1000.setTexture(tex1000[1]);
                        drawSprites(window,whichStage,drawRaiseButtons);
                        window.display();
                    }
                }
                if(canIGo(2000)&&drawRaiseButtons==1)
                {
                    if(x>=1100&& x<=1185&&y>=625&&y<=659)       ///2000
                    {
                        spr2000.setTexture(tex2000[0]);
                        drawSprites(window,whichStage,drawRaiseButtons);
                        window.display();
                        if(event.type==Event::MouseButtonPressed)
                        {
                            controller=2000;
                            return;
                        }
                    }
                    else
                    {
                        spr2000.setTexture(tex2000[1]);
                        drawSprites(window,whichStage,drawRaiseButtons);
                        window.display();
                    }
                }
            }
            else
            {
                setMyTexturesDeafult();
                drawSprites(window,whichStage,drawRaiseButtons);
                window.display();
            }
        }while(window.pollEvent(event));
    }
    return;
}
int letAIDecide(int howMuchToGoIn,int whosTurn,int whichStage)
{
    int controller,chance;  ///chance od 0 do 10
    if(rand()%9==0)///blefuje
    {
        while(controller +howMuchToGoIn>player[whosTurn].totalMoney)
        {
            controller=(player[whosTurn].totalMoney-howMuchToGoIn)*(rand()%5)/4;
            if(controller==0)
                return 2;
            else
                return controller;
        }
    }
    else
    {
        chance=chanceToWin(fillCardsTo7(player[whosTurn].card1,player[whosTurn].card2,whichStage),whichStage,playersInRound())/10;
        if(chance<3&&howMuchToGoIn!=0)    ///folduje
            return 1;
        else
        {
            if(chance<=4)   /// wchodzi lub nie w zaleznosci od wysokosci stawki
            {
                if(howMuchToGoIn*15>player[whosTurn].totalMoney*chance)
                    return 1;       ///folduje
                else
                    return 2;   ///calluje
            }
            else
            {
                if(chance <=8)
                {
                    return player[whosTurn].totalMoney*chance/10;
                }
                else
                    return -1;
            }
        }
    }
}
void bidding(RenderWindow &window, int whoStarts,int startingPot, int &moneyOnTable, const int whichStage,const int smallBlind,const int bigBlind)
{
    int whosTurn=whoStarts;
    bool endBidding=false;
    int pot=startingPot;
    int atLeastOne=0;
    setAllMoneyTextures(moneyOnTable);
    drawSprites(window,whichStage,drawRaiseButtons);
    window.display();
    int controller;
    int tempPlayersInRound=playersInRound();
    while(endBidding==false)
    {
        while(player[whosTurn].inRound==false|| player[whosTurn].isAllIn==true)///jak trafi na nieaktywnego gracza lub all-in  to przesuwa dalej
        {
            whosTurn=(whosTurn+1)%7;
        }
        atLeastOne++;

        if(whosTurn==0) ///tu decyduje gracz
        {
            if(player[0].totalMoney==0)
                player[0].isAllIn=true;
            if(player[0].isAllIn==false)
            {
                eventService(window,7000,whichStage,whosTurn,controller,pot,smallBlind,bigBlind);
                if(controller==1) ///fold
                {
                    moneyOnTable+=player[whosTurn].moneyIn;
                    player[whosTurn].moneyIn=0;
                    player[whosTurn].inRound=false;
                }
                if(controller==2) ///check
                {
                    player[whosTurn].goIn(pot-player[whosTurn].moneyIn);
                    if(player[0].totalMoney==0)
                        player[0].isAllIn=true;

                }
                if(controller>2) ///raise
                {
                    if(controller+pot-player[0].moneyIn>=player[0].totalMoney)
                        controller=-1;
                    else
                    {
                        player[whosTurn].goIn(controller+pot-player[0].moneyIn);
                        pot+=controller;
                    }
                }
                if(controller==-1) ///all in
                {
                    player[whosTurn].goAllIn();
                    if(pot<player[whosTurn].moneyIn)
                        pot=player[whosTurn].moneyIn;
                }
            }
        }

        else ///tu decyduje komputer
        {

            setMyTexturesDeafult();
            player[whosTurn].sprAvatar.setTexture(player[whosTurn].tex0);
            drawSprites(window,whichStage,drawRaiseButtons);
            window.display();
            eventService(window,rand()%3+1,whichStage,whosTurn,controller,pot,smallBlind,bigBlind);
            controller=letAIDecide(pot-player[whosTurn].moneyIn,whosTurn,whichStage);
            if(controller>=player[whosTurn].totalMoney)
                controller=-1;
            if(controller==1) ///fold
            {
                moneyOnTable+=player[whosTurn].moneyIn;
                player[whosTurn].moneyIn=0;
                player[whosTurn].inRound=false;
            }
            if(controller==2) ///check
            {
                player[whosTurn].goIn(pot-player[whosTurn].moneyIn);

            }
            if(controller>2) ///raise
            {
                controller=controller/smallBlind;
                controller=controller*smallBlind;
                if(controller==0)
                    controller=smallBlind;
                if(controller+pot-player[whosTurn].moneyIn>=player[whosTurn].totalMoney)
                    controller=-1;
                else
                {
                    player[whosTurn].goIn(controller+pot-player[whosTurn].moneyIn);
                    pot+=controller;
                }
            }
            if(controller==-1) ///all in
            {
                player[whosTurn].goAllIn();
                if(pot<player[whosTurn].moneyIn)
                    pot=player[whosTurn].moneyIn;
            }
            if(player[whosTurn].inRound==true)
                player[whosTurn].sprAvatar.setTexture(player[whosTurn].tex1);
            else
                player[whosTurn].sprAvatar.setTexture(player[whosTurn].tex2);
            drawSprites(window,whichStage,drawRaiseButtons);
            window.display();
        }
        setAllMoneyTextures(moneyOnTable);
        endBidding=true;
        for(int i=0;i<7;i++)
        {
            if(!(player[i].moneyIn==pot || player[i].inRound==false || player[i].isAllIn==true))
                endBidding=false;
        }
        if(playersInRound()==1)
            break;
        if(atLeastOne+allInPlayers()<tempPlayersInRound)
            endBidding=false;
        whosTurn=(whosTurn+1)%7;
    }

    for(int i=0;i<7;i++)
    {
        moneyOnTable+=player[i].moneyIn;
        player[i].moneyIn=0;
    }
}
void round( RenderWindow &window, int whoStarts,int smallBlind, int bigBlind)
{
    int whosTurn=whoStarts;
    player[whosTurn].goIn(smallBlind);
    whosTurn=(whosTurn+1)%playersInRound();
    player[whosTurn].goIn(bigBlind);
    whosTurn=(whosTurn+1)%playersInRound();
    int pot =bigBlind;
    int moneyOnTable=0;
    drawSprites(window,0,drawRaiseButtons);      ///rysuje moje 2 karty
    window.display();
    bidding(window,whosTurn,pot,moneyOnTable,0,smallBlind,bigBlind);      ///licytacja 1    ///tu
    if(playersInRound()==1)
    {
        int i=0;
        for( ;i<7;i++)
        {
            if(player[i].inRound==true)
                break;
        }
        player[i].totalMoney+=moneyOnTable;
        return;
    }
    drawSprites(window,1,drawRaiseButtons);      ///pokazuje flopa
    window.display();
    bidding(window,whosTurn,0,moneyOnTable,1,smallBlind,bigBlind);       ///licytacja 2
    if(playersInRound()==1)
    {
        int i=0;
        for( ;i<7;i++)
        {
            if(player[i].inRound==true)
                break;
        }
        player[i].totalMoney+=moneyOnTable;
        return;
    }
    drawSprites(window,2,drawRaiseButtons);          ///pokazuje turna
    window.display();
    bidding(window,whosTurn,0,moneyOnTable,2,smallBlind,bigBlind);       ///licytacja 3
    if(playersInRound()==1)
    {
        int i=0;
        for( ;i<7;i++)
        {
            if(player[i].inRound==true)
                break;
        }
        player[i].totalMoney+=moneyOnTable;
        return;
    }
    drawSprites(window,3,drawRaiseButtons);          ///pokazuje rivera
    window.display();
    bidding(window,whosTurn,0,moneyOnTable,3,smallBlind,bigBlind);       ///licytacja 4
    if(playersInRound()==1)
    {
        int i=0;
        for( ;i<7;i++)
        {
            if(player[i].inRound==true)
                break;
        }
        player[i].totalMoney+=moneyOnTable;
        return;
    }
    drawSprites(window,4,drawRaiseButtons);          ///pokazuje karty grajacych
    window.display();
    int a;
    eventService(window,7,4,1,a,1,1,1);
    dealMoneyToWinners(moneyOnTable);
    eventService(window,2,4,1,a,1,1,1);
                ///funkcja ktora wylania zwyciezce
}
int increaseBigBlind(int roundsCounter)
{
    int bigBlind;
    if(roundsCounter>=10)///zwiekszanie blindow
    {
        bigBlind=200;
        if(roundsCounter>=20)
        {
            bigBlind=500;
            if(roundsCounter>=30)
            {
                bigBlind=1000;
                if(roundsCounter>=40)
                    bigBlind=2000;
            }
        }
    }

    return bigBlind;
}
void kickLoosers(int bigBlind)
{
    for(int i=0;i<7;i++)
    {
        if(player[i].totalMoney<bigBlind)
        {
            player[i].inGame=false;
            player[i].totalMoney=0;
        }

    }
}
void endGameScreen(RenderWindow &window)
{
    Event event;
    int x,y;
    window.clear(Color::White);
    window.draw(sprGameOver);
    window.draw(sprNewGame);
    window.display();
    while(1)
    {
        window.clear(Color::White);
        window.draw(sprGameOver);
        window.draw(sprNewGame);
        window.display();
        while(window.pollEvent(event))
        {
            window.clear(Color::White);
            window.draw(sprGameOver);
            window.draw(sprNewGame);
            window.display();
            if(event.type== Event::Closed)
                window.close(),exit(0);
            if(event.type== Event::Closed)
                    window.close();
            x=Mouse::getPosition(window).x;
            y=Mouse::getPosition(window).y;
            if(x>520&&x<640&&y>580&&y<630)
            {
                sprNewGame.setTexture(texNewGame[1]);
                if(event.type==Event::MouseButtonPressed)
                    return;
            }
            else
                sprNewGame.setTexture(texNewGame[0]);
        }
    }
}
int main()
{

    srand(time(0));
    loadTextures();
    loadCards();
    declarePlayers();
    RenderWindow window( VideoMode(1200, 800,32 ), "poker" );
    while(1)
    {
        int whoStarts=5;
        int smallBlind=50,bigBlind=100,roundsCounter=1;
        while(playersInGame()>1&&player[0].inGame==true)
        {///trzeba pomyslec jeszcze nad all inami- zeby komp wiedzial ze to all in
            drawing();
            setSpritePositions();
            setPlayersInRound();
            round(window,whoStarts,smallBlind, bigBlind);
            whoStarts=(whoStarts+1)%playersInGame();
            if(roundsCounter&10==0)     ///zwieksz blindy
                smallBlind=bigBlind,bigBlind=increaseBigBlind(roundsCounter);
            roundsCounter++;
            kickLoosers(bigBlind);
        }
        endGameScreen(window);
    }


    return 0;
}
