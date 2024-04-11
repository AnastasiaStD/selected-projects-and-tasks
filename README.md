# selected-projects-and-tasks
Это реализация консольного приложения,
позволяющего применять к изображениям различные фильтры,
аналогичные фильтрам в популярных графических редакторах.

## Поддерживаемый формат изображений

Входные и выходные графические файлы должны быть в формате [BMP](http://en.wikipedia.org/wiki/BMP_file_format).

Формат BMP поддерживает достаточно много вариаций, но в этом задании будет использоваться
24-битный BMP без сжатия и без таблицы цветов. Тип используемого `DIB header` - `BITMAPINFOHEADER`.

Пример файла в нужном формате есть в статье на Википедии [в разделе "Example 1"](https://en.wikipedia.org/wiki/BMP_file_format#Example_1)
и в папке [test_script/data](test_script/data).

При тестировании обязательно обращайте внимание на то, чтобы тестовое изображение
было сохранено именно в 24-битном BMP.

## Формат аргументов командной строки

Описание формата аргументов командной строки:

`{имя программы} {путь к входному файлу} {путь к выходному файлу}
[-{имя фильтра 1} [параметр фильтра 1] [параметр фильтра 2] ...]
[-{имя фильтра 2} [параметр фильтра 1] [параметр фильтра 2] ...] ...`

При запуске без аргументов программа выводит справку.

### Пример
`./image_processor input.bmp /tmp/output.bmp -crop 800 600 -gs -blur 0.5`

В этом примере
1. Загружается изображение из файла `input.bmp`
2. Обрезается до изображения с началом в верхнем левом углу и размером 800х600 пикселей
3. Переводится в оттенки серого
4. Применяется размытие с сигмой 0.5
5. Полученное изображение сохраняется в файл `/tmp/output.bmp`

Список фильтров может быть пуст, тогда изображение должно быть сохранено в неизменном виде.
Фильтры применяются в том порядке, в котором они перечислены в аргументах командной строки.

## Фильтры

В формулах далее считаем, что каждая компонента цвета
представлена вещественным числом от 0 до 1. Цвета пикселей
представлены тройками `(R, G, B)`. Таким образом, `(0, 0, 0)` – черный,
`(1, 1, 1)` – белый.

Если фильтр задан матрицей, это означает, что значение каждого из цветов определяется взвешенной суммой
значений этого цвета в соседних пикселях в соответствии с матрицей. При этом целевому пикселю
соответствует центральный элемент матрицы.

### Список базовых фильтров

#### Crop (-crop width height)
Обрезает изображение до заданных ширины и высоты. Используется верхняя левая часть изображения.

Если запрошенные ширина или высота превышают размеры исходного изображения, выдается доступная часть изображения.

#### Grayscale (-gs)
Преобразует изображение в оттенки серого по формуле

![encoding](https://latex.codecogs.com/svg.image?R'%20=%20G'%20=%20B'%20=0.299%20R%20&plus;%200%20.587%20G%20&plus;%200%20.%20114%20B)

#### Negative (-neg)
Преобразует изображение в негатив по формуле

![encoding](https://latex.codecogs.com/svg.image?R'%20=%201%20-%20R,%20G'%20=%201%20-%20G,%20B'%20=%201%20-%20B)

#### Sharpening (-sharp)
Повышение резкости. Достигается применением матрицы

![encoding](https://latex.codecogs.com/svg.image?%5Cbegin%7Bbmatrix%7D%20&%20-1%20&%20%20%5C%5C-1%20&%205%20&%20-1%20%5C%5C%20&%20-1%20&%20%20%5C%5C%5Cend%7Bbmatrix%7D)

#### Edge Detection (-edge threshold)
Выделение границ. Изображение переводится в оттенки серого и применяется матрица

![encoding](https://latex.codecogs.com/svg.image?%5Cbegin%7Bbmatrix%7D%20&%20-1%20&%20%20%5C%5C-1%20&%204%20&%20-1%20%5C%5C%20&%20-1%20&%20%20%5C%5C%5Cend%7Bbmatrix%7D)

Пиксели со значением, превысившим `threshold`, окрашиваются в белый, остальные – в черный.

#### Gaussian Blur (-blur sigma)
[Гауссово размытие](https://ru.wikipedia.org/wiki/Размытие_по_Гауссу),
параметр – сигма.

Значение каждого из цветов пикселя `C[x0][y0]` определяется формулой

![encoding](https://latex.codecogs.com/svg.image?C%5Bx_0%5D%5By_0%5D%20%3D%20%5Csum_%7Bx%3D0%2Cy%3D0%7D%5E%7Bwidth-1%2C%20height-1%7DC%5Bx%5D%5By%5D%5Cfrac%7B1%7D%7B2%5Cpi%5Csigma%5E2%7De%5E%7B-%5Cfrac%7B%5Cleft%7Cx_o-x%5Cright%7C%5E2%20%26plus%3B%20%5Cleft%7Cy_o-y%5Cright%7C%5E2%7D%7B2%5Csigma%5E2%7D%7D)

Существуют различные варианты релализации и оптимизации вычисления этого фильтра, описание есть [в Википедии](https://ru.wikipedia.org/wiki/Размытие_по_Гауссу).

### Дополнительный фильтр

Дополнительно надо сделать один собственный фильтр.


