# Video-Games-Sales
Repository for data analysis for Video Games Sales dataset
This is my first repository in GitHub. 
If you are starting at Data Analytics or Data Science, you will probably get to know the Video Games Sales dataset. It is really common to find different versions of this dataset online at Kaggle.
I tried to use my own inspiration to clean out the data, to do the plots, to show the variables I believed where the most intresting ones.
Obviously there will be mistakes and lots of other statistics can be carried out. this was just a first approach.

Please read my MEDIUM story for this code.
Regards,
Domingo
----------------------------------------------
< CODE >
import pandas as pd
import matplotlib as mpl
import matplotlib.pyplot as plt
import seaborn as sns

df= pd.read_csv("data/Video_Games_Sales_as_at_22_Dec_2016.csv")
df

![image.png](attachment:image.png)


df.dtypes

df.shape

df.info()

df.describe()

numericas= []
for i in df.columns:
    if df[i].dtypes== "float":
        numericas.append(i)

categoricas= []
for i in df.columns:
    if df[i].dtypes== "object":
        categoricas.append(i)

print(categoricas, numericas)

history= df.sort_values(by=['Year_of_Release'])
history.head(15)

news= history.loc[lambda df: df['Year_of_Release'] >= 2015]

news.head(15)

news.tail(20)

### Primero hablemos de los nullos.

### Veamos cuantos nulls hay en las distintas variables
import missingno as msno
%matplotlib inline
msno.matrix(df.sample(250))

msno.bar(df)

Vemos que en todo lo que son las variables de los críticos hay muchos blancos. De hecho hasta casi la mitad de las muestras!

### Primero actuemos sobre los que faltan de YEAR. Descartemoslos.  
df= df.dropna(subset=["Year_of_Release"])

### Actualizamos el cuadro de arriba y ahora quedaron algunos blancos en Name, en Genre y en Publisher. Eliminemos el de Name ya que es un juego sin nombre...
df= df.dropna(subset= ["Name"])

df

### Quiero ver como aparecen los vacíos en las columnas que tienen faltantes
df[["Critic_Count", "Critic_Count", "User_Score", "User_Count", "Developer", "Rating"]].head(40)

history[["Critic_Count", "Critic_Count", "User_Score", "User_Count", "Developer", "Rating"]].head(40)

We can see there are plenty of rows where all this columns are NaN, mainly to the oldest games!
Our solution will be to set a threshold, where a column must have at least 11 columns with values, if not, it will drop. 
Let´s create a new dataset in order not to loose our df.


df.shape

df_con_resena= df.dropna(thresh= 11)
df_con_resena

### ¡ ME quede con 9940 filas ! Veamos el grafico de los nulls denuevo.
msno.bar(df_con_resena)

Ahora quiero llenar los ausentes en las críticas, y podemos llenarlo así. A modo diccionario

![image.png](attachment:image.png)

Critic Score: suele ser un número entre 0 y 100.
Critic Count: lo mismo. Cantidad de críticos.
User Score: 0 a 10. QUE ES TBD?? LIMPIAR
User Count: infinito. Aplicamos media.
Developer: Puedo llenar "No developer"
Rating: VER. Limpiar.

df_con_resena.info()

df_con_resena["Rating"].unique()

![image.png](attachment:image.png)

Por lo tanto reemplazamos los nulos por RP que es Rating Pending!!!

df_con_resena["User_Score"].unique()

tbd is to be determined! Let´s fill that Na = tbd

## Lets fill NAs !
values= {"Publisher": "No publisher Identified", 
         "Critic_Score": df_con_resena["Critic_Score"].mean(),
         "Critic_Count": df_con_resena["Critic_Count"].median(),
         "User_Score": "tbd", 
         "User_Count": df_con_resena["User_Count"].median(), 
         "Developer": "no developer", 
         "Rating": "RP"}
df_con_resenaR= df_con_resena.fillna(value= values)
df_con_resenaR

if df_con_resenaR["User_Score"]== "tbd":
    pd.count()

## Veamos como quedan los blancs para este nuevo dataFrame rellenado (de ahí la R del final del nombre)
msno.bar(df_con_resenaR)

### Perfecto!!! Todo lleno

categoricas

for i in categoricas:
    print(df[i].unique())

Veo que salvo las categorías de Genre y Platform, el resto tienen muchos unique values. 

catsGrafic= categoricas[1:3]
catsGrafic

import warnings
warnings.simplefilter(action='ignore', category=FutureWarning)

import seaborn as sns
for i in catsGrafic:
  draw = sns.countplot(df[i])
  draw.set_xticklabels(draw.get_xticklabels(), rotation="vertical", ha="right")
  plt.show()

import seaborn as sns
for i in catsGrafic:
  draw = sns.countplot(df_con_resenaR[i])
  draw.set_xticklabels(draw.get_xticklabels(), rotation="45", ha="right")
  plt.show()

###  Acción y Deportes liderando siempre!

### Play2 y Xbox 360 las dos consolas más populares

Vamos a hacer también un count para el año!


sns.set(rc={'figure.figsize':(11.7,8.27)})
draw= sns.countplot(df["Year_of_Release"],  palette="summer")
draw.set_xticklabels(draw.get_xticklabels(), rotation="vertical", ha="right")
plt.show()

df["Year_of_Release"].describe()

50% of the data is between 2003 and 2010.

### Pasemos ahora  a las numéricas. El boxplot no me gusta, veamos la distribución.

numericas

for i in numericas:
    draw= sns.histplot(df[i], bins= 10, kde= True)
    draw.set_xticklabels(draw.get_xticklabels(), rotation="vertical")
    plt.show()

i´m not satisfied with the way the sales where graphed.

From a Kaggle collegue i found out the following function to plot in a more organized way.

def grafDistr(data, column, range_min=0, range_max=0, clr='blue'):
    print(f'1% percentile: {data[column].quantile(.1)}')
    print(f'50% percentile: {data[column].quantile(.50)}')
    print(f'99% percentile: {data[column].quantile(.99)}')  # me tira en texto a que altura está el 1%, el 50% y el 99%
  
    data1 = data.copy()
    fig = plt.figure(figsize=(15, 6)) 
    if range_min==range_max:
        range_min = data[column].min()
        range_max = data[column].max()
    else:
        data1 = data1[data1[column]<data1[column].quantile(range_max)]
        data1 =  data1[data1[column]>data1[column].quantile(range_min)]
    
    sns.histplot(data1[column], kde=True, bins=len(data1[column].unique()), color=clr)
    plt.title(f"{column} histogram")
    plt.ylabel(f"{column} count")
    plt.xlabel(column)

    plt.show()

grafDistr(df, "NA_Sales", range_max=0.99)

grafDistr(df, "EU_Sales", range_max=0.99)

grafDistr(df, "JP_Sales", range_max=0.99)

grafDistr(df, "Global_Sales", range_max=0.99)

## Chequiemos a que se refiere con global sales, haciendo una nueva columna TOTAL SALES, sumando todos los datos que tenemos
df["total_sales"]= df["NA_Sales"] + df["EU_Sales"] + df["JP_Sales"]+ df["Other_Sales"]
df

### ¡Remember the sales are in millions!

grafDistr(df, "total_sales", range_max=0.99)
## Vemos que son casi lo mism que Global_Sales 

![image.png](attachment:image.png)

Who are more demanding? Critics or users? Let's see the distribution

Let´s go back to our filled df.

grafDistr(df_con_resenaR, "Critic_Score", clr= "red")

The high bar is because we filledNA with median! Remember!

## Agrego también el sin relleno para ver que el critic score ya tenía un pico en la mediana. 
draw1= grafDistr(df_con_resena, "Critic_Score", clr= "red")

grafDistr(df_con_resenaR, "User_Score", clr= "green")
### vemos que en user_score tenemos el tbd. Por lo tanto no me deja ver la distrib

### The above error is because, as we show below, between the user_score we have tbd, making the whole column an "object". We must drop this values and graph 

df_con_resenaR.User_Score.unique()

user3= df_con_resenaR.loc[df_con_resenaR.User_Score != "tbd"]
user3["User_Score"]= user3["User_Score"].astype("float")

user3.dtypes

## Ahora que desechamos los tbd. Hagamos un count plot de los user_score
draw2= grafDistr(user3,"User_Score",clr="green")

We can see the main values for User´s score are higher than Critic Score

# -----------------------------------------------------------------------------------------------------------------

### Now let´s try to see sales by Platform

#### For that we will use the total_sales and the timeline. Grouping by Platform

df_con_resenaR.head()


df1= df_con_resenaR.reset_index()
df1

df1= df1.drop(axis= 1, columns= "index")

df1

Para lo que yo quiero hacer, tendría que agrupar por YearofRelease, agrupar por Platform, y después sumar las GlobalSales, y UserCounts

Hagamos un df separado con estas cols

df_group= df1.loc[:, ["Year_of_Release", "Platform", "Global_Sales", "User_Count"]]
df_group

## sum = df.groupby(["year", "month"]).agg({"score": "sum", "num_attempts": "sum"}).reset_index()
grupi= df_group.groupby(["Year_of_Release", "Platform"]).agg({"Global_Sales": "sum", "User_Count": "sum"}).reset_index()

## Otra forma de hacerlo.
### df_sales_by_year = df.pivot_table(index = ['year',
                                          # 'platform'], 
                                  #values = 'sum_sales', 
                                  #aggfunc = 'sum').reset_index()
### df_sales_by_year

grupi["Year_of_Release"]= grupi["Year_of_Release"].astype("int")

grupi

grupi["Platform"].unique()

fig, ax = plt.subplots(figsize= (12,10))

grouped = grupi.groupby('Platform')
for key, group in grouped:
    group.plot(ax=ax, kind='line', x='Year_of_Release', y='Global_Sales', label=key)
plt.show()

This is the central idea of the graph. It is not that nice... too much lines and legends.

Let´s do a bar graph as the one we made with COUNT.

We can also see the year most videogames were sold. 

sns.set_theme(style="whitegrid")
sns.barplot(x= "Platform", y= "Global_Sales", data= grupi, palette="Blues_d", capsize= 0)

grupi

grupi.groupby('Platform')['Platform'].count().sort_values(ascending = False)

grupi.groupby("Platform")["Global_Sales"].sum().sort_values(ascending= False)

## Best Platform= PS2 

best_year= grupi.groupby("Year_of_Release")["Global_Sales"].sum().sort_values(ascending=False)

best_year

## Best Year= 2008 

### All this belonged to the df_con_reseñaR, let´s try to do it with df, our initial dataframe
## sum = df.groupby(["year", "month"]).agg({"score": "sum", "num_attempts": "sum"}).reset_index()
df_grouped= df.groupby(["Year_of_Release", "Platform"]).agg({"Global_Sales": "sum", "User_Count": "sum"}).reset_index()

best_total_platform= df_grouped.groupby("Platform")["Global_Sales"].sum().sort_values(ascending= False)
best_total_platform

best_total_year= df_grouped.groupby("Year_of_Release")["Global_Sales"].sum().sort_values(ascending=False)
best_total_year

### Best Platform is still PS2 and best year is still 2008 

Primero grafiquemos en el tiempo únicamente a los que tienen más de 200 millones de ventas

def lineas(data, categ_column, x, y, lines, labels=['','','']):
    
    fig = plt.figure(figsize=(15, 6)) 
    for i in lines:
        plot_data = data[data[categ_column]==i]
        plt.plot( 
            plot_data[x],
            plot_data[y], 
            linestyle= '-', 
             marker='o',
            linewidth=2, 
            alpha=0.9,
            )
    plt.grid(True)
    plt.title(labels[0])
    plt.xlabel(labels[1])
    plt.ylabel(labels[2])
    plt.legend(lines)
    plt.show()

## More than 200 mill Sales

labels = ['Total sales of TOP Platforms with more than 200 million by year','Year','Number of games sold, in millions']
lineas(grupi, 
                'Platform', 
                'Year_of_Release', 
                'Global_Sales', 
                ['PS2', "X360", "PS3", "Wii", "DS", "PS4", "PS", "XB", "PSP", "PC"],
               labels)

## From 2000 to 2015. 

siglo21= grupi.loc[grupi["Year_of_Release"]>= 2000]

labels = ['Total sales of TOP Platforms 21st Century','Year','Number of games sold, in millions']
lineas(siglo21, 
                'Platform', 
                'Year_of_Release', 
                'Global_Sales', 
                ['PS2', "X360", "PS3", "Wii", "DS", "PS4", "PS", "XB", "PSP", "PC"],
               labels)

## Let´s do a comparison between the main companies.

siglo21["company"]= siglo21["Platform"]

siglo21["company"]= siglo21["company"].replace(to_replace={"XB":"Microsoft", "X360":"Microsoft", "XOne":"Microsoft", "PC":"Microsoft", "DC": "Nintendo",
                                                          "PS": "PlayStation", "PS2": "PlayStation", "PS3": "PlayStation", "PS4": "PlayStation", "PSP":"PlayStation",
                                                          "PSV":"PlayStation", "GBA":"Nintendo", "DS": "Nintendo", "GC":"Nintendo", "3DS":"Nintendo",
                                                           "GB":"Nintendo", "Wii":"Nintendo", "WiiU":"Nintendo"}, value= None)

There are 3 big companies that span all the VideoGames Platforms. This are PlayStation (for all the PS prototypes), Nintendo (for all the GameBoys and Wii), and Microsoft( owner of Xbox and Xbox 360). I also considered the PC games as part of Microsoft!

Notice we add DC (sega) to Nintendo since this company owns most of its property rights. 

siglo21

siglo21.groupby("company")["Global_Sales"].sum().sort_values(ascending= False)

compas= siglo21.groupby(["Year_of_Release", "company"]).agg({"Global_Sales": "sum"}).reset_index()
compas

Nint= compas.loc[compas["company"]== "Nintendo"]
PS= compas.loc[compas["company"]== "PlayStation"]
Mic= compas.loc[compas["company"]=="Microsoft"]

maxnin= Nint["Global_Sales"].max()
maxPs= PS["Global_Sales"].max()
maxMic= Mic["Global_Sales"].max()

print(maxnin)
print(maxPs)
print(maxMic)

fig, ax = plt.subplots(figsize=(20,12))
ax.plot(Nint["Year_of_Release"], Nint["Global_Sales"], label="Nintendo Sales", linewidth= 4)
ax.plot(PS["Year_of_Release"], PS["Global_Sales"], label="PlayStation Sales", linewidth= 4)
ax.plot(Mic["Year_of_Release"], Mic["Global_Sales"], label="Microsoft Sales", linewidth= 4)
ax.axhline(maxnin, linestyle="--", color= "b")
ax.axhline(maxPs, linestyle="--", color="orange")
ax.axhline(maxMic, linestyle="--", color="g")
ax.set_title("Sales by Company, in millions", size= "20", fontweight= "bold" )
ax.set_ylabel("Sales", size= "16")
ax.set_xlabel("Year", size= "16")
ax.legend(prop={'size': 16})
plt.show()


The graph show the highest sales for each company. Being 2008 for Nintendo, 2004 for PlayStation, and 2010 for Microsoft.

This results are not convincing to me... I believe that if we went back to the first dataset with the blanks the conclusions can be different. 

# df was our first dataset. Lets group by year, platform and global sales as we did. 
siglo21bis= df.loc[df["Year_of_Release"]>= 2000]

grupi2= siglo21bis.groupby(["Year_of_Release", "Platform"]).agg({"Global_Sales":"sum"}).reset_index()

grupi2

grupi2["company"]= grupi2["Platform"]
grupi2["company"]= grupi2["company"].replace(to_replace={"XB":"Microsoft", "X360":"Microsoft", "XOne":"Microsoft", "PC":"Microsoft", "DC": "Nintendo",
                                                          "PS": "PlayStation", "PS2": "PlayStation", "PS3": "PlayStation", "PS4": "PlayStation", "PSP":"PlayStation",
                                                          "PSV":"PlayStation", "GBA":"Nintendo", "DS": "Nintendo", "GC":"Nintendo", "3DS":"Nintendo",
                                                           "GB":"Nintendo", "Wii":"Nintendo", "WiiU":"Nintendo"}, value= None)

grupi2

grupi3= grupi2.groupby(["Year_of_Release", "company"]).agg({"Global_Sales":"sum"}).reset_index()

labels = ['Total sales of TOP Platforms 21st Century','Year','Number of games sold, in millions']
lineas(grupi3, 
                'company', 
                'Year_of_Release', 
                'Global_Sales', 
                ["Nintendo", "PlayStation", "Microsoft"],
               labels)

grupi3.groupby("company")["Global_Sales"].sum().sort_values(ascending= False)

My hipothesis is FALSE. The Global_Sales patterns are the same one with the clean dataset and with the initial one. 

--------------------------------------------------------------------------

# Football Games

deportes= df.loc[df["Genre"]== "Sports"]
deportes

soccer= deportes[deportes['Name'].str.contains("FIFA|PES|Soccer")]
soccer

270 ROWS!!!


game= soccer.groupby("Name")["Global_Sales"].sum().sort_values(ascending= False)
game

game.head(10)

## Fifa 15 Leads in Sales 

game_platform= soccer.groupby(["Name", "Platform"])["Global_Sales"].sum().sort_values(ascending= False)
game_platform

game_platform.head(10)

### But look at that! If you split platforms, FIFA 16 for PS4 is the one with highest Sales 

game_critic= soccer.groupby(["Name", "Platform"])["Critic_Score"].sum().sort_values(ascending= False)

game_critic.head(10)

game_critic.tail(10)

## I want to see the lowest critics different to 0.
deportes_with_critic= df_con_resenaR.loc[df["Genre"]== "Sports"]
soccer_with_critic= deportes_with_critic[deportes_with_critic['Name'].str.contains("FIFA|PES|Soccer")]


soccer_with_critic.loc[soccer_with_critic["Critic_Score"]<=30]

soccer


soccer_TOP= soccer.loc[(soccer["Name"]== "FIFA 15")| (soccer["Name"]== "FIFA 16")| (soccer["Name"]== "FIFA 17")|
                      (soccer["Name"]== "Winning Eleven: Pro Evolution Soccer 2007")|  (soccer["Name"]== "FIFA Soccer 06")]

![image.png](attachment:image.png)

soccer_TOP

soccer_TOP_group= soccer_TOP.groupby(["Name", "Platform"])["JP_Sales", "NA_Sales", "EU_Sales", "Other_Sales"].sum().sort_values(by= "Name", ascending= True)

soccer_TOP_group

soccer_TOP_group.reset_index()

soccer_sales= soccer_TOP.groupby(["Name"])["JP_Sales", "NA_Sales", "EU_Sales", "Other_Sales"].sum().sort_values(by= "Name", ascending= True)

soccer_sales.reset_index()

We can see how Winning Eleven was the king in JAPAN and in North America it sold very little. 

FIFA 15 and 16 really got into NA and EU being above the competition

top_games= ['FIFA 15',
  'FIFA 16',
  'FIFA 17',
  'Winning Eleven: Pro Evolution Soccer 2007',
  'FIFA Soccer 06']

import seaborn as sns
sns.set_theme(style="whitegrid")
ax = sns.barplot(x="Name", y="JP_Sales", hue= "Platform", data=soccer_TOP)
ax.set_title("Sales in JAPAN for Top Games")
plt.show()

### This graph is not good to show what we wanna show.

import seaborn as sns
sns.set_theme(style="whitegrid")
ax = sns.barplot(x=soccer_sales.index, y="JP_Sales", data=soccer_sales, palette="autumn")
ax.set_title("Sales in JAPAN for Top Games")
plt.show()

ax = sns.barplot(x=soccer_sales.index, y="NA_Sales", data=soccer_sales, palette="autumn")
ax.set_title("Sales in North America for Top Games")
plt.show()

ax = sns.barplot(x=soccer_sales.index, y="EU_Sales", data=soccer_sales, palette="autumn")
ax.set_title("Sales in Europe for Top Games")
plt.show()

ax = sns.barplot(x=soccer_sales.index, y="Other_Sales", data=soccer_sales, palette="autumn")
ax.set_title("Sales in Rest of the World for Top Games")
plt.show()



soccer_sales.index
