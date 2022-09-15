<table style="width: 100%;">
  <tr>
    <td style="text-align: center; border: none;">
    Министерство образования и науки РФ<br>
Государственное бюджетное профессиональное образовательное учреждение Республики Марий Эл<br>
Йошкар-Олинский технологический колледж
</td>
  </tr>
  <tr>
    <td style="text-align: center; border: none; height: 15em;">
    <h2 style="font-size:3em;">MySQL,Desktop,Wpf</h2>
 <h2 style="font-size:3em;">    <br> Разработка desktop-приложений<br><br> <b> </b></h2> </h3></td>
  </tr>
  <tr>
    <br><br><td style="text-align: right; border: none; height: 20em;">
      Разработал: <br/>
      Попов Илья
      Группа: И-31<br>
     
   
  </tr>
  <tr>
    <td style="text-align: center; border: none; height: 5em;">
    г.Йошкар-Ола,<br> 2022</td>
  </tr>
</table>

# Оглавление


1.[Создание подключения к БД MySQL](#создание-подключения-к-бд-mysql)

2.[Вывод данных согласно макету (ListView).](#вывод-данных-согласно-макету-listview)

3.[Пагинация,поиск,сортировка,фильтрация](#пагинацияпоисксортировкафильтрация)

3.1[Пагинация](#пагинация)

3.2[Поиск](#поиск)

3.3[Сортировка](#сортировка)

3.4[Фильтрация](#фильтрация)

4[Invalidate,INotifyPropertyChanged](#invalidateinotifypropertychanged)

3.6[Получение списка типов материала](#получение-списка-типов-материала)

4.[Добавление и редактирование материалов](#добавление-и-редактирование-материалов)







# Создание подключения к БД MySQL
## Получение данных из базы

 ### 1.Создаем интерфейс поставщика данных IDataProvider(в интерфейсе пишем)

 ```cs 
 // Materials-это класс,GetMaterials-это метод
 IEnumerable<Materials> GetMaterials();
 ```
 ### 2.Создаем класс MySqlDataProvider, реализующий этот интерфейс
```cs 
//MySQLDataProvider реализует наш интерфейс IDataProvider

class MySQLDataProvider: IDataProvider
{
    // соединение с базой данных,MySqlConnection-установите пакет MySQl.Data 
    private MySqlConnection Connection;
```
### 3. Создаем модель "Materials"(класс и заполняем его ,так же как и в бд)
```cs
//класс должен быть публичным public,иначе будет ошибка
public class Materials
    {
        public int ID { get; set; }
        public string NameMaterial { get; set; }
        public int MaterialType_ID { get; set; }
        public string Image { get; set; }
        public decimal Price { get; set; }
        public int QuantityInSclad { get; set; }
        public int MinQuantity { get; set; }
        public int QuantityInPack { get; set; }
        public string Edizm { get; set; }
        public string MaterialTypeTitle { get; set; }
    }
     
 ````
 ### 4.В классе MySqlDataProvider создаем конструктор
 ```cs
  public MySQLDataProvider()
        {
            try
            {//пишем ваш сервер у меня 192.168.1.32,имя базы у меня user49 и пароль у меня 55405,charset-utf8 -это кодировка 
                Connection = new MySqlConnection("Server=192.168.1.32;port=3306;Database=user49;UserId=user49;password=55405;charset=utf8;");
            }
            catch (Exception ex)
            {

                MessageBox.Show(ex.Message);
            }

        }

```
### 5.Реализуем метод GetMaterials(нажав на IDataProvider-релизовать интерфейс )
```cs
 public IEnumerable<Materials> GetMaterials()
        {//в нем создаем лист
          List<Materials> MaterialList = new List<Materials>();
//сдесь пишем запрос ,который выведет материалы и типы материалов
          string Query = @"select m.*,
                        mt.Title AS MaterialTypeTitle
                        -- ,s.Materials_ID
                        from Materials m
                        left join MaterialType mt on mt.ID=m.MaterialType_ID;";
         try          
            {//GetMaterialTypes();
                // открываем соединение с сервером
                Connection.Open();
                try
                {
                    // создаем команду
                    MySqlCommand Command = new MySqlCommand(Query, Connection);
                    // получаем результат команды (массив строк)
                    MySqlDataReader Reader = Command.ExecuteReader();

                    // перебираем стоки
                    while (Reader.Read())
                    {
                        // создаем экземпляр класса 
                        Materials NewProduct = new Materials();
                        // и заполняем его поля
                        NewProduct.ID = Reader.GetInt32("ID");
                        NewProduct.NameMaterial = Reader.GetString("NameMaterial");
                        NewProduct.Edizm = Reader.GetString("Edizm");
                        NewProduct.MinQuantity = Reader.GetInt32("MinQuantity");
                        NewProduct.QuantityInPack = Reader.GetInt32("QuantityInPack");
                        NewProduct.QuantityInSclad = Reader.GetInt32("QuantityInSclad");

                        NewProduct.Image = Reader["Image"].ToString();

                         NewProduct.MaterialTypeTitle = Reader["MaterialTypeTitle"].ToString();

                        NewProduct.Price = Reader.GetInt32("Price");

                        // добавляем экземпляр класса в список продуктов
                        MaterialList.Add(NewProduct);
                    }
                }
                finally
                {
                    // обязательно закрываем соединение
                    // ресурсы сервера конечны
                    Connection.Close();
                }
            }
            catch (Exception)
            {
            }

            return MaterialList;

        }
```

### 6.Создаем класс  Globals
```cs
class Globals
{
    public static IDataProvider DataProvider;
}
```
    
### 7. В конструкторе главного окна(MainWindow) создаем поставщика данных и получаем с помощью него список материалов   
```cs
public IEnumerable<Materials> MaterialList { get; set; }
        public MainWindow()
        {
            InitializeComponent();
            DataContext = this;
            Globals.DataProvider = new MySQLDataProvider();
            MaterialList = Globals.DataProvider.GetMaterials();
           
        }

```

# Вывод данных согласно макету (ListView).

## 1.В гриде разметки главного окна пишем
```cs
<Grid>


<ListView
    Grid.Row="1"
    Grid.Column="1"
    ItemsSource="{Binding MaterialList}"
>
//чтобы растягивалась плитка
   <ListView.ItemContainerStyle>
                <Style 
            TargetType="ListViewItem">
                    <Setter 
                Property="HorizontalContentAlignment"
                Value="Stretch" />
                </Style>
            </ListView.ItemContainerStyle>
 //
 //Внутри него вставляем макет для элемента списка
<ListView.ItemTemplate>
    <DataTemplate>
        <Border 
            BorderThickness="1" 
            BorderBrush="Black" 
            CornerRadius="0">
//Внутри макета вставляем Grid из трёх колонок: для картинки, основного содержимого и стоимости.
          
                        <Grid 
    Margin="10" 
    HorizontalAlignment="Stretch">

                            <Grid.ColumnDefinitions>
                                <ColumnDefinition Width="64"/>
                                <ColumnDefinition Width="*"/>
                                <ColumnDefinition Width="auto"/>
                            </Grid.ColumnDefinitions>
//фотография материала
<Image
    Width="64" 
    Height="64"
    Source="{Binding ImagePreview,TargetNullValue={StaticResource defaultImage}}" />
    
 <StackPanel
    Grid.Column="1"
    Margin="5"
    Orientation="Vertical">

                                <TextBlock 
Text="{Binding TypeAndName}" FontFamily=" Segoe Print" FontSize="14"/>

                                <TextBlock 
 Text="{Binding MinCountT}" FontFamily=" Segoe Print" FontSize="14"/>


        </StackPanel>
           </Grid>
          </Border>
        </DataTemplate>
  </ListView.ItemTemplate>    
 </Grid>
</ListView>
```
### 2.В классе Materials пишем
```cs

public Uri ImagePreview
        {
            get
            {
                var imageName = Environment.CurrentDirectory + (Image ?? "");
                return System.IO.File.Exists(imageName) ? new Uri(imageName) : null;
            }
        }
//гетеры коротрые мы указали в разметке Text="{Binding TypeAndName}"
public string TypeAndName
        {
            get
            {
                return CurrentMaterialType.Title + " | " + NameMaterial;
            }
        }
        public string MinCountT
        {
            get
            {
                return " Минимальное количество: " + MinQuantity;
            }
        }
 ```
 ### Изображение по-умолчанию задается в ресурсах окна (перед гридом)
  ```xml
  <Window.Resources>
        <BitmapImage 
        x:Key='defaultImage' 
        UriSource='./Image/picture.png' />
</Window.Resources>
```

# Пагинация,поиск,сортировка,фильтрация
## Пагинация
```xml
//Создаем в нижней панели WrapPanel и кидаем 5 или 4 Label и обрабатываем клик на них
//(&lt; -это < )(&gt;- это >)
<WrapPanel  Grid.Row="2"HorizontalAlignment="Right" Width="149"  Grid.Column="1">
            <Label Content="&lt;" FontFamily=" Century Gothic" FontSize="20" MouseDown="Label_MouseDown"/>

            <Label Content="1" FontFamily=" Century Gothic" FontSize="20" MouseDown="Label_MouseDown_1"/>

            <Label Content="2" FontFamily=" Century Gothic" FontSize="20" MouseDown="Label_MouseDown_2"/>

            <Label Content="3" FontFamily=" Century Gothic" FontSize="20" MouseDown="Label_MouseDown_3"/>
            <Label Content="4" FontFamily=" Century Gothic" FontSize="20" MouseDown="Label_MouseDown_4"/>

            <Label Content="&gt;" FontFamily=" Century Gothic" FontSize="20" MouseDown="Label_MouseDown_5"/>
        </WrapPanel>

```
### Заводим переменую  CurrentPage 
```cs 
 public int CurrentPage = 1;
```
### Обрабатываем клики по кнопке
```cs
  private void Label_MouseDown(object sender, MouseButtonEventArgs e)
        {
            if (CurrentPage > 1)
            {
                CurrentPage--;
                Invalidate();
            }
        }

        private void Label_MouseDown_1(object sender, MouseButtonEventArgs e)
        {
            CurrentPage = 1;
            Invalidate();
        }

        private void Label_MouseDown_2(object sender, MouseButtonEventArgs e)
        {
            CurrentPage = 2;
            Invalidate();
        }

        private void Label_MouseDown_3(object sender, MouseButtonEventArgs e)
        {
            CurrentPage = 3;
            Invalidate();
        }

        private void Label_MouseDown_4(object sender, MouseButtonEventArgs e)
        {
            CurrentPage = 4;
            Invalidate();
        }

        private void Label_MouseDown_5(object sender, MouseButtonEventArgs e)
        {
            CurrentPage++;
            Invalidate();
        }
```
### И в гетере добавляем 
 ```cs
 //у меня по 15 страниц
 return Result.Skip((CurrentPage - 1) * 15).Take(15);
 ```
 # Поиск
 ### 1.В верхнюю панель добавляем текстовое поле для ввода строки поиска
```xml
<Label 
    Content="Поиск" 
    VerticalAlignment="Center"/>
<TextBox
    Width="200"
    VerticalAlignment="Center"
    x:Name="SearchFilterTextBox" 
    KeyUp="SearchFilterTextBox_KeyUp"/>
 ```
 ###  2.В коде окна запоминаем вводимую строку

```cs

private string SearchFilter="";
private void SearchFilterTextBox_KeyUp(object sender, KeyEventArgs e)
{
    SearchFilter = SearchFilterTextBox.Text;
    Invalidate();
}
```
## Гетер (Правим гетер)
```cs
private IEnumerable<Materials> _MaterialList;

public IEnumerable<Materials> MaterialList{
    get {
        var Result = _MaterialList;
//это фильрация
      /*  if (ProductTypeFilterId > 0)
            Result = Result.Where(i => i.ProductTypeID == ProductTypeFilterId);
*/
/*Сортировка
        switch (SortType)
        {
            // сортировка по названию продукции
            case 1:
                Result = Result.OrderBy(p => p.Title);
                break;
            case 2:
                Result = Result.OrderByDescending(p => p.Title);
                break;
                // остальные сортировки реализуйте сами

        }
*/
        //Вот поиск
        if (SearchFilter != "")
            Result = Result.Where(
                p => p.Title.IndexOf(SearchFilter, StringComparison.OrdinalIgnoreCase) >= 0 ||
                        p.Description.IndexOf(SearchFilter, StringComparison.OrdinalIgnoreCase) >= 0
            );

       

        return Result;
    } 
    set {
        _MaterialList = value;
        Invalidate();
    }
}
```
# Сортировка
### Создаем массив со списком типов сортировок
 ```cs 
 public string[] SortList { get; set; } = {
    "Без сортировки",
    "Наименование по убыванию",
    "Наименование по возрастанию",
    "Остаток на складе по убыванию",
    "Остаток на складе по возрастанию",
    "Цена по убыванию",
    "Цена по возрастанию" };
```
### В разметке добавляем элемент выпадающий список (ComboBox)
 ```xml
 <Label 
        Content="Сортировка: "
        VerticalAlignment="Center"/>
            <ComboBox
        Name="SortTypeComboBox"
        SelectedIndex="0"
        VerticalContentAlignment="Center"
        MinWidth="200"
        SelectionChanged="SortTypeComboBox_SelectionChanged"
        ItemsSource="{Binding SortList}"/>
```

### Реализуем обработчик выбора из списка
#### Запоминаем ИНДЕКС выбранного элемента
 
 
```cs
private int SortType = 0;

    private void SortTypeComboBox_SelectionChanged(object sender, SelectionChangedEventArgs e)
            {
                SortType = SortTypeComboBox.SelectedIndex;
                Invalidate();
            }

```
### И дорабатываем геттер

```cs

        switch (SortType)
        {
           // сортировка по названию 
                    case 1:
                        Result = Result.OrderBy(p => p.NameMaterial);
                        break;
                    case 2:
                        Result = Result.OrderByDescending(p => p.NameMaterial);
                        break;
                    case 3:
                        Result = Result.OrderBy(p => p.QuantityInSclad);
                        break;
                    case 4:
                        Result = Result.OrderByDescending(p => p.QuantityInSclad);
                        break;
                    case 5:
                        Result = Result.OrderBy(p => p.Price);
                        break;
                    case 6:
                        Result = Result.OrderByDescending(p => p.Price);
                        break;

        }
```
# Invalidate,INotifyPropertyChanged
```cs
//реализуем интерфейс 
 public partial class MainWindow : Window, INotifyPropertyChanged
    {
         
public event PropertyChangedEventHandler PropertyChanged;
//сдесь ваш лист,у меня MaterialList
private void Invalidate(string ComponentName = "MaterialList")
        {
            if (PropertyChanged != null)
                PropertyChanged(this, new PropertyChangedEventArgs(ComponentName));
        }
 ```


# Фильтрация
# Получение списка типов материала
### Создаем класс MaterialType
```cs
public class MaterialType
    {
        public int ID { get; set; }
        public string Title { get; set; }
        public override string ToString()
        {
            return Title;
        }
    }

```
### В IDataProvider пишем

```cs
  IEnumerable<MaterialType> GetMaterialTypes();
```
### в Globals добавляем
```cs
 public static IEnumerable<MaterialType> MaterialTypeList { get; set; }
```

### Реализуем метод  GetMaterialTypes


```cs
private List<MaterialType> ProductTypes = null;

  public IEnumerable<MaterialType> GetMaterialTypes()
        {
            if (ProductTypes == null)
            {
                ProductTypes = new List<MaterialType>();

              
                string Query = "SELECT * FROM MaterialType";

                try
                {
                    Connection.Open();
                    try
                    {
                        MySqlCommand Command = new MySqlCommand(Query, Connection);
                        MySqlDataReader Reader = Command.ExecuteReader();

                        while (Reader.Read())
                        {
                            MaterialType NewProductType = new MaterialType();
                            NewProductType.ID = Reader.GetInt32("ID");
                            NewProductType.Title = Reader.GetString("Title");

                            ProductTypes.Add(NewProductType);
                        }
                    }
                    finally
                    {
                        Connection.Close();
                    }
                }
                catch (Exception)
                {
                }
             

            }
            return ProductTypes;

        }
```
### Также добавляем
```cs
private MaterialType GetProductType(int Id)
        {
            // тут заполнится список типов продукции, если он ещё пустой
            GetMaterialTypes();
            return ProductTypes.Find(pt => pt.ID == Id);
        }
```





### Создаем список материалов, заполняем его данными из базы и добавляем в начало пункт "Все  типы"
 ```cs
public List<MaterialType> MaterialTypeList { get; set; }

 public MainWindow()
        {
            InitializeComponent();
           /* DataContext = this;
            Globals.DataProvider = new MySQLDataProvider();
            MaterialList = Globals.DataProvider.GetMaterials();
            */
            MaterialTypeList = Globals.DataProvider.GetMaterialTypes().ToList();
            MaterialTypeList.Insert(0, new MaterialType { Title = "Все типы" });

        }
```
### В разметке в верхнюю панель добавляем выпадающий список
```xml
<Label Content="Тип продукции" VerticalAlignment="Center" Width="97"/>

<ComboBox
    Width="132"
    x:Name="ProductTypeFilter"
    VerticalAlignment="Center"
    SelectedIndex="0"
    SelectionChanged="ProductTypeFilter_SelectionChanged"
    ItemsSource="{Binding MaterialTypeList}" Height="28"/>
```
### Реализуем обработчик выбора элемента фильтра и дорабатываем гетер
 ```cs
 private int ProductTypeFilterId = 0;

        private void ProductTypeFilter_SelectionChanged(object sender, SelectionChangedEventArgs e)
        {
            // запоминаем ID выбранного типа
            ProductTypeFilterId = (ProductTypeFilter.SelectedItem as MaterialType).ID;
            Invalidate();
        }




   get {
        var Result = _MaterialList;
//это фильрация
       if (ProductTypeFilterId > 0)
            Result = Result.Where(i => i.ProductTypeID == ProductTypeFilterId);
 ```

# Добавление и редактирование материалов

1. Создайте новое окно: **EditWindow**

2. В классе окна **EditWindow** добавьте свойство *CurrentMaterial*, в котором будет храниться добавляемый/редактируемый экземпляр материала:

    ```cs
   public Materials CurrentMaterial { get; set; }
    ```

    И геттер для названия окна:

    ```cs
    public string WindowName
        {
            get
            {
                return CurrentMaterial.ID == 0 ? "Новый продукт" : "Редактирование продукта";
            }
        }
    ```
3. В конструктор окна добавьте параметр типа **Materials** и присвойте его ранее объявленному свойству:

    ```cs
   public EditWindow(Materials p)
        {
            InitializeComponent();
            DataContext = this;
            CurrentMaterial = p;
            ProductTypes = Globals.DataProvider.GetMaterialTypes();
        }
    ```

4. В разметке окна вместо фиксированного названия вставьте привязку к свойству *WindowName*

    ```xml
    <Window
        ...
        Title="{Binding WindowName}">
    ```

5. В окне создайте сетку из трёх колонок: в первой у нас будет изображение, во второй редактируемые поля продукта, а в третей список материалов

    ```xml
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="auto"/>
        <ColumnDefinition Width="*"/>
        <ColumnDefinition Width="auto"/>
    </Grid.ColumnDefinitions>
    ```

6. Во вторую колонку добавьте **StackPanel** с границами (чтобы визуальные компоненты не прилипали к границам окна) и в этом списке разместите редактируемые элементы

    

    ```xml
      <StackPanel 
    Margin="5">
            <Label Content="Наименование материала" FontFamily=" Segoe Print" FontSize="14"/>
            <TextBox Text="{Binding CurrentMaterial.NameMaterial}" FontFamily=" Segoe Print" FontSize="14"/>

            <Label Content="Цена" FontFamily=" Segoe Print" FontSize="14"/>
            <TextBox Text="{Binding CurrentMaterial.Price}" FontFamily=" Segoe Print" FontSize="14"/>

     

    </StackPanel>
    ```

    

    * Выбор типа материала из списка

        В классе окна объявляем свойство *ProductTypes* - список типов продукции

        ```cs
            public IEnumerable<MaterialType> ProductTypes { get; set; }
        ```

        И в конструкторе получаем его из поставщика данных

        ```cs
       ProductTypes = Globals.DataProvider.GetMaterialTypes();
        ```

       

        В поставщик данных добавим переменную, в которой будет храниться список типов продукции:

        ```cs
        private List<MaterialType> ProductTypes = null;
        ```


        В модели **Materials** убираем свойства *MaterialTypeTitle*, *MaterialType_ID* и добавляем *CurrentMaterialType* 

        ```cs
       public MaterialType CurrentMaterialType { get; set; }
        ```

        И в классе **MySQLDataProvider** переделайте получение типа материала:

        ```cs
        //добавте GetMaterialTypes(); в метод получения материалов перед открытием соединения
          NewProduct.CurrentMaterialType = GetProductType(Reader.GetInt32("MaterialType_ID"));
        ```

        Реализация метода *GetProductType*:

        ```cs
       private MaterialType GetProductType(int Id)
        {
            // тут заполнится список типов продукции, если он ещё пустой
            GetMaterialTypes();
            return ProductTypes.Find(pt => pt.ID == Id);
        }
        ```

        Теперь в верстке окна редактирования продукции мы можем использовать выпадающий список

        ```xml
        <ComboBox 
            ItemsSource="{Binding ProductTypes}"
            SelectedItem="{Binding CurrentMaterial.CurrentMaterialType}"/>
        ```

    * смена изображения продукции

        Вывод изображения производится как и в главном окне

        ```xml
        <Image
            Width="200" 
            Height="200"
            Source="{Binding CurrentMaterial.ImagePreview,TargetNullValue={StaticResource defaultImage}}" />
        ```

        А для смены изображения используем стандартный диалог Windows, повесив его на кнопку *Сменить картинку* (кнопку добавьте сами в **StackPanel**)

        Обработчик кнопки:

        ```cs
        private void ChangeImage_Click(object sender, RoutedEventArgs e)
        {
              OpenFileDialog GetImageDialog = new OpenFileDialog();
              GetImageDialog.Filter = "Файлы изображений: (*.png, *.jpg)|*.png;*.jpg";
              GetImageDialog.InitialDirectory = Environment.CurrentDirectory;
             if (GetImageDialog.ShowDialog() == true)
            {
                image.Source = new BitmapImage(new Uri(GetImageDialog.FileName));
                CurrentMaterial.Image = GetImageDialog.FileName.Substring(Environment.CurrentDirectory.Length);
                Invalidate();
            }
        }
        ```

    * Многострочное описание

        Тут просто - разрешаем переносы и задаем высоту элемента

        ```xml
        <Label Content="Описание материала"/>
        <TextBox 
            AcceptsReturn="True"
            Height="2cm"
            Text="{Binding CurrentMaterial.Description}"/>
        ```

7. Сохранение введенных данных

    В разметку добавьте кнопку **Сохранить** и напишите обработчик

    ```cs
    private void Button_Click(object sender, RoutedEventArgs e)
    {
        // вся работа с БД должна быть завернута в исключения
            try
            {
                // сюда добавлять проверки

                // метод SaveProduct реализуем ниже
                Globals.DataProvider.SaveMaterial(CurrentMaterial);
                DialogResult = true;
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
    }
    ```

    В интерфейсе **IDataProvider** объявляем метод *SaveProduct*:

    ```cs
     void SaveMaterial(Materials currentMaterial);
    ```

    И реализуем его в классе поставщика данных 

    ```cs
   
            public void SaveMaterial(Materials currentMaterial)
            {
                Connection.Open();
                try
                {
                    if (currentMaterial.ID == 0)
                    {
                        // новый продукт - добавляем запись
                        string Query = @"INSERT INTO Materials
            (NameMaterial,
            MaterialType_ID,
            Image,
            Price,
            QuantityInSclad,
            MinQuantity,
            QuantityInPack,
            Edizm)
            VALUES
            (@NameMaterial,
            @MaterialType_ID,
            @Image,
            @Price,
            @QuantityInSclad,
            @MinQuantity,
            @QuantityInPack,
            @Edizm)";

                        MySqlCommand Command = new MySqlCommand(Query, Connection);
                        Command.Parameters.AddWithValue("@NameMaterial", currentMaterial.NameMaterial);
                        Command.Parameters.AddWithValue("@MaterialType_ID", currentMaterial.CurrentMaterialType.ID);
                        Command.Parameters.AddWithValue("@Image", currentMaterial.Image);
                        Command.Parameters.AddWithValue("@Price", currentMaterial.Price);
                        Command.Parameters.AddWithValue("@QuantityInSclad", currentMaterial.QuantityInSclad);
                        Command.Parameters.AddWithValue("@MinQuantity", currentMaterial.MinQuantity);
                        Command.Parameters.AddWithValue("@QuantityInPack", currentMaterial.QuantityInPack);
                        Command.Parameters.AddWithValue("@Edizm", currentMaterial.Edizm);
                        Command.ExecuteNonQuery();
                    }
                    else
                    {
                        // существующий продукт - изменяем запись

                        string Query = @"UPDATE Materials
                                    SET
                                    NameMaterial = @NameMaterial,
                                    MaterialType_ID = @MaterialType_ID,
                                    Image = @Image,
                                    Price = @Price,
                                    QuantityInSclad = @QuantityInSclad,
                                    MinQuantity = @MinQuantity,
                                    QuantityInPack = @QuantityInPack,
                                    Edizm = @Edizm
                                    WHERE ID = @ID";


                        MySqlCommand Command = new MySqlCommand(Query, Connection);
                        Command.Parameters.AddWithValue("@NameMaterial", currentMaterial.NameMaterial);
                        Command.Parameters.AddWithValue("@MaterialType_ID", currentMaterial.CurrentMaterialType.ID);
                        Command.Parameters.AddWithValue("@Image", currentMaterial.Image);
                        Command.Parameters.AddWithValue("@Price", currentMaterial.Price);
                        Command.Parameters.AddWithValue("@QuantityInSclad", currentMaterial.QuantityInSclad);
                        Command.Parameters.AddWithValue("@MinQuantity", currentMaterial.MinQuantity);
                        Command.Parameters.AddWithValue("@QuantityInPack", currentMaterial.QuantityInPack);
                        Command.Parameters.AddWithValue("@Edizm", currentMaterial.Edizm);
                        Command.Parameters.AddWithValue("@ID", currentMaterial.ID);
                        Command.ExecuteNonQuery();
                    }
                }
                finally
                {
                    Connection.Close();
                }
            }
    ```

8. Открытие окна редактирования для существующей и новой продукции

    * для редактирования существующей продукции в списке продукции реализуем обработчик двойного клика

        ```cs
         private void ListView_MouseDoubleClick(object sender, MouseButtonEventArgs e)
        {
            var NewEditWindow = new Windows.EditWindow(Ls.SelectedItem as Materials);
            if ((bool)NewEditWindow.ShowDialog())
            {
                
                MaterialList = Globals.DataProvider.GetMaterials();
            }
        }
        ```

    * для создания нового материала в разметке главного окна создайте кнопку "Добавить материал" (либо в верхней панели, либо в левой) и в её обработчике создайте новый экземпляр продукта

        ```cs
       var NewEditWindow = new Windows.EditWindow(new Materials());
            NewEditWindow.ShowDialog();
            MaterialList = Globals.DataProvider.GetMaterials();
      
        ```
