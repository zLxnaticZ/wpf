# POS Exam - C# WPF 

## 0 Connections
## 0.1 Scope Definition
##Moritz vs mine
```csharp
    xmlns:vm="clr-namespace:Calculator.ViewModel"
    xmlns:validation="clr-namespace:Registration.Validation"
```

    xmlns:local="clr-namespace:BMI"
    xmlns:vm="clr-namespace:BMI.ViewModels"
```

## 0.2 View Model - View
```csharp
    <Window.DataContext>
        <vm:MainViewModel />
    </Window.DataContext>
```

## 0.3 Converter as Resource (insert validation rule, converter etc.)
```csharp
##Moritz vs mine

    <Window.Resources>
        <con:DateConverter x:Key="DateConverter" />
    </Window.Resources>
    
    <Window.Resources>
        <vm:MinMaxValidation x:Key="MinMaxValidation"/>
    </Window.Resources>
```

## 1 Layouts
### 1.1 Grid
```csharp
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="160"/>
            <ColumnDefinition />
        </Grid.ColumnDefinitions>
        <Grid.RowDefinitions>
            <RowDefinition />
            <RowDefinition />
        </Grid.RowDefinitions>
        <TextBlock Grid.Row="0" Grid.Column="0">Test</TextBlock>
        <TextBlock Grid.Row="0" Grid.Column="1">Test</TextBlock>
        <TextBlock Grid.Row="1" Grid.Column="0">Test</TextBlock>
        <TextBlock Grid.Row="1" Grid.Column="1">Test</TextBlock>
    </Grid>
```

### 1.2 Stack Panel
vertical positioning
```csharp
    <StackPanel>
        ...
    </StackPanel>
```

## 2 Controls
### 2.1 TextBox
- text as user input / text input field
```csharp
    <TextBox />
```

### 2.2 TextBlock
- just a simple label
```csharp
    <TextBlock />
```

### 2.3 Slider
```csharp
    <Slider Minimum="0" Maximum="255" Margin="0,0,0,20" Value="{Binding R, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" />
```

### 2.4 DatePicker
```csharp
    <DatePicker Grid.Row="5" Grid.Column="1" SelectedDate="{Binding FirstSchoolDay, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged, ValidatesOnDataErrors=True}" HorizontalAlignment="Left" Margin="0,10.4,0,0" VerticalAlignment="Top" Width="385" Height="27"/>
```

### 2.5 RadioButton
```csharp
    <StackPanel Grid.Row="6" Grid.Column="1" Margin="0,46.4,0.2,0.2" >
        <RadioButton GroupName="Gender" Command="{Binding ChangeGender}" CommandParameter="MALE">Male</RadioButton>
        <RadioButton GroupName="Gender" Command="{Binding ChangeGender}" CommandParameter="FEMALE">Female</RadioButton>
        <RadioButton GroupName="Gender" Command="{Binding ChangeGender}" CommandParameter="OTHER">Other</RadioButton>
    </StackPanel>
```

### 2.6 ComboBox
```csharp
    <ComboBox FontSize="36" SelectedIndex="0" Margin="0,0,0,10" SelectedValue="{Binding OpCode, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" HorizontalContentAlignment="Center" SelectedValuePath="Content">
        <ComboBoxItem>+</ComboBoxItem>
        <ComboBoxItem>-</ComboBoxItem>
        <ComboBoxItem>*</ComboBoxItem>
        <ComboBoxItem>/</ComboBoxItem>
    </ComboBox>
```

### 2.7 Menu
- if menu item is clicked => trigger relay command defined in VM (= view model)
```csharp
    <Menu>
        <MenuItem Header="File">
            <MenuItem Header="Open" Command="{Binding OpenFileCommand}" />
            <Separator />
            <MenuItem Header="Exit" Command="{Binding ExitCommand}" />
        </MenuItem>
        <MenuItem Header="Help">
            <MenuItem Header="About" Command="{Binding AboutCommand}" />
        </MenuItem>
    </Menu>
```

## 3 Binding
### 3.1 Property change
```csharp
    private void UpdateProperty(string propName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propName));
    }
```
example:
```csharp
    UpdateProperty("Result");
```
> Update in setters!

### 3.2 Property
```csharp
    public int A {
        get => _a;
        set
        {
            _a = value;
            UpdateProperty("Result");
            UpdateProperty("A");
        }
    }
```

## 3.3 XAML
- `Binding <PropertyName>`
- `Mode=TwoWay` if view should be able to manipulate/set the property
- `UpdateSourceTrigger=PropertyChanged`
- `ValidatesOnDataErrors=True` used together with IDateErrorInfo

### 3.3.1 XAML bindings as attributes
```csharp
    "{Binding FirstSchoolDay, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged, ValidatesOnDataErrors=True}"
```

### 3.3.1 XAML bindings as elements
- used together with validation rules
```csharp
    <TextBox Grid.Row="2" Grid.Column="1" Background="{DynamicResource {x:Static SystemColors.HotTrackBrushKey}}" Foreground="White" Margin="0,46.8,-0.8,46.8">
        <TextBox.Text>
            <Binding Path="Password" Mode="TwoWay" UpdateSourceTrigger="PropertyChanged" ValidatesOnDataErrors="True">
                <Binding.ValidationRules>
                    <validation:PasswordValidation />
                </Binding.ValidationRules>
            </Binding>
        </TextBox.Text>
    </TextBox>
```

## 4 Commands
- use `RelayCommand.cs` => implements `ICommand`
```csharp
    public ICommand ExitCommand { get; }
    ...
    public MainViewModel()
    {
        ExitCommand = new RelayCommand(
            () =>
            {
                System.Windows.Application.Current.Shutdown();
            }
        );
        ...
    }
```

## 5 Converters
- implement `IValueConverter`
- `public object Convert(object value, Type targetType, object parameter, CultureInfo culture)` converts Property to View Type
- `public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)` converts View Type to Property
```csharp
    class DateConverter : IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            if(value is DateTime)
            {
                DateTime dateTime = (DateTime)value;
                return $"{dateTime.Day}/{dateTime.Month}/{dateTime.Year}";
            }
            return null;
        }

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            string input = (string)value;
            try
            {
                var cultureInfo = new CultureInfo("en-US");
                return DateTime.Parse((string)value);
            }
            catch (Exception e)
            {

            }
            return null;
        }
    }
```


## Gabriel Date Time converter class
```
namespace RegistrationForm.ViewModels
{
    class DateTimeConverter : IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, System.Globalization.CultureInfo culture)
        {
            if (value != null)
            {
                DateTime test = (DateTime)value;
                string date = test.ToString("dd/MM/yyyy");
                return (date);
            }
            return string.Empty;
        }

        public object ConvertBack(object value, Type targetType, object parameter, System.Globalization.CultureInfo culture)
        {
            if (DateTime.TryParseExact((string)value, "dd/MM/yyyy", null, System.Globalization.DateTimeStyles.None, out DateTime result)) {
                return result;
            }
            return null;
        }
    }
}
```



## 6 Validation
## Moritz vs mine

- implement `ValidationRule`
- method: `public override ValidationResult Validate(object value, CultureInfo cultureInfo)`
```csharp
    class CharacterValidation : ValidationRule
    {
        public override ValidationResult Validate(object value, CultureInfo cultureInfo)
        {
            var val = (string)value;
            if(Regex.Match(val, @"^[A-Za-z]*$").Success)
            {
                return new ValidationResult(true, null);
            }
            return new ValidationResult(false, "input contains invalid characters");
        }

    }
    ```
    
    
    
    public class MinMaxValidation : ValidationRule
    {
        public int Min { get; set; }
        public int Max { get; set; }

        public override ValidationResult Validate(object value, CultureInfo cultureInfo)
        {
            int number;
            if (!int.TryParse((string)value, out number))
            {
                return new ValidationResult(false, "Not a number");
            }
            if (number < Min)
            {
                return new ValidationResult(false, "Too low");
            }
            if (number > Max)
            {
                return new ValidationResult(false, "Too high");
            }
            return ValidationResult.ValidResult;
        }
    }
```



# Random Sliders, Textboxes, Textblocks, Labels, Dates with Bindings, nested and MinMaxValidation
```
<Slider Grid.Column="1" Value="{Binding Height}" Minimum="0" Maximum="300" HorizontalAlignment="Left" Margin="5,0,0,0" VerticalAlignment="Center" Width="385"/>
        <Slider HorizontalAlignment="Left" Value="{Binding Weight}" Minimum="0" Maximum="150" Margin="5,24,0,0" VerticalAlignment="Top" Width="385" Grid.Column="1" Grid.Row="1"/>
        <Label Content="Height" HorizontalAlignment="Center" VerticalAlignment="Center"/>
        <Label Content="Weight" HorizontalAlignment="Center" VerticalAlignment="Center" Grid.Row="1"/>
        <Label Content="Birthdate" HorizontalAlignment="Center" VerticalAlignment="Center" Grid.Row="2"/>
        <Label Content="Result" HorizontalAlignment="Center" VerticalAlignment="Center" Grid.Row="3"/>
        <TextBlock Grid.Column="1" HorizontalAlignment="Left" Margin="10,0,0,0" Grid.Row="3" Text="{Binding Result}" TextWrapping="Wrap" VerticalAlignment="Center"/>
        <DatePicker Grid.Column="1" SelectedDate="{Binding Birthdate}" HorizontalAlignment="Left" Margin="5,0,0,0" Grid.Row="2" VerticalAlignment="Center"/>
        <TextBox HorizontalAlignment="Left" Margin="270,0,0,0" TextWrapping="Wrap" VerticalAlignment="Center" Width="120">
            <TextBox.Text>
                <Binding Path="Height" UpdateSourceTrigger="PropertyChanged">
                <Binding.ValidationRules>
                    <vm:MinMaxValidation Min="0" Max="300"/>
                </Binding.ValidationRules>
                </Binding>
            </TextBox.Text>
        </TextBox>
        <TextBox HorizontalAlignment="Left" Margin="270,24,0,0" TextWrapping="Wrap" VerticalAlignment="Top" Width="120" Grid.Row="1" TextChanged="TextBox_TextChanged">
            <TextBox.Text>
                <Binding Path="Weight" UpdateSourceTrigger="PropertyChanged">
                <Binding.ValidationRules>
                    <vm:MinMaxValidation Min="0" Max="150"/>
                </Binding.ValidationRules>
                </Binding>
            </TextBox.Text>
        </TextBox>
```


###Combobox gender example in MainViewmModel
```

private List<string> _genders = new (){"male", "female", "other"};
        private string _selectedGender = "other";

        public string SelectedGender
        {
            get => _selectedGender;
            set
            {
                if (value == _selectedGender) return;
                _selectedGender = value;
                OnPropertyChanged(nameof(SelectedGender));
                OnPropertyChanged(nameof(Result));
            }
        }

        public List<string> Genders
        {
            get => _genders;
            set
            {
                if (Equals(value, _genders)) return;
                _genders = value;
                OnPropertyChanged(nameof(Genders));
            }
        }
        ```
        
        
        
##In xaml
```
<ComboBox SelectedItem="{Binding SelectedGender}" ItemsSource="{Binding Genders}" HorizontalAlignment="Center" Margin="0,37,0,0" Grid.Row="4" VerticalAlignment="Top"   
Width="120" />
```
```



###Relay close program, constructor

public MainViewModel()
        {
            CloseCommand = new RelayCommand<string>(par =>
            {
                System.Windows.Application.Current.Shutdown();
            });
        }
    

     ###Close command
        public ICommand CloseCommand { get; set; }



### Menu close program example
<Menu Grid.ColumnSpan="2" Margin="0,0,0,46">
     <MenuItem Header="Program">
           <MenuItem Header="Close" Command="{Binding CloseCommand}" />
     </MenuItem>
</Menu>




# Don't forget
- TwoWay bindings
- Trigger property changes
- validate binding attribute at IDateErrorInfo XAML
- IDataErrorInfo => trigger other property changes
