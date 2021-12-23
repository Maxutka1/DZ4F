Условие задачи:

Удалить заданную вершину и в модифицированном графе отсортировать вершины по убыванию степени вершины.

Выполнение: 

Код программы: 

//Подключаем стандартные библиотеки СИ.
#include <stdio.h>
#include <stdlib.h>

/*
Программа завершилась с ошибкой.
*/
#define ERROR       -1

/*
Программа завершилась успешно.
*/
#define SUCCSESS    0

//Истина
#define TRUE    1
//Ложь
#define FALSE   0

//Максимальное число узлов в графе. квадрат числа должен быть меньше 2^32
#define MAX_VALUE   (int)((1u << 15u) - 1u)

//Имя файла в который будет запсанна информация о графе на языке DOT.
#define GRAPH_FILE_NAME         "Graph.txt"

#define NEW_GRAPH_FILE_NAME     "NewGraph.txt"

//Имя файла в который будет записанно изображение графа.
#define GRAPH_IMAGE_FILE_NAME       "Graph.png"

#define NEW_GRAPH_IMAGE_FILE_NAME   "NewGraph.png"

//Максимальная длинна системной команды.
#define COMMAND_MAX_LENGTH  1000

/*
Функция отправляет системе команду на создание файла с изображением графа.
graph_file_name - файл в котором граф записан на языке DOT
image_file_name - файл в который будет записанно изображение
*/
void ConvertGraphToPNG(const char* graph_file_name, const char* image_file_name);

/*
Функция генерирует текстовый файл в который на языке DOT будет записанна информация о графе.

file_mane - имя файла в который будет записан граф
adjacency_matrix - матрица инцедентности в виде которой хранится граф в нутри программы
matrix_size - размер матрицы инцедентности (количество столбцов равно количеству строк и равно размеру матрицы)
deleting_node - номер удалённого узла (нужен для правильного вывода номеров вершин)
*/
void CreateGraphFile(const char* file_name, int* adjacency_matrix, int matrix_size, int deleting_node);

/*
Функция Удаляет указанный узел из графа.
adjacency_matrix - исходный граф представленный в виде таблицы инцедентности. В случае успешного выполнения функции будет заменён на новый. При этом номера всех узлов стоящих после удалённого уменьшатся на единицу.
matrix_size - размер исходного графа. В случае успешного выполнения функции будет уменьшен на единицу.
node_number - номер удаляемого узла
*/
void DeleteNode(int** adjacency_matrix, int* matrix_size, int node_number);

/*
Функция возвращяет номер элемента в массиве, в виде которого хранится квадратная матрица.

matrix_size - размер матрицы
row_mnumber - номер строки в которой хранится элемент
column_number - номер столбца в котором хранится элемент
*/
int GetId(int matrix_size, int row_number, int column_number);

/*
Функция записывает информацию считанную из консоли в матрицу инцедентности.

adjacency_matrix - массив в который будет записана информация
nodes_count - количество узлов во входящем графе
*/
void GetGraphFromConsole(int* adjacency_matrix, int nodes_count);

/*
Функция возвращяет степень указанной вершишы, которая находится в указанном графе.

adjacency_matrix - массив, хранящий информацию о графе в виде матрицы инцедентности
nodes_count - количество узлов в графе
node_number - номер узла для которого проверяется степень
*/
int GetNodeDegree(const int* adjacency_matrix, int nodes_count, int node_number);

/*
Функция записывает в массив узлов, упорядоченных по убыванию их степени.
nodes_list - указатель на заранее созданный список. После завершения функции данный массив будет содержать список узлов упорядоченных по убыванию их степени.
adjacency_matrix - граф, для которого будет генерироваться список
nodes_count - количество узлов в графе (соответственно количество элементов в массиве nodes_list)
*/
void GetSortedNodesList(int* nodes_list, const int* adjacency_matrix, int nodes_count);

int main(void)
{
    int nodes_count;//количество узлов в графе

    printf("Set count of nodes.\n");//Сообщяем пользователю о том, что хотим считать количество узлов в графе.

    scanf("%i", &nodes_count);//получаем количество узлов в новом графе

    if (nodes_count <= 0)                   //Если указанное количество узлов меньше 0, то
    {
        printf("Count of nodes is low.\n"); //выводим сообщение об ошибке и
        exit(ERROR);                        //завершаем работу программы.
    }

    if (nodes_count > MAX_VALUE)    //Если количество узлов в графе больше максимально допустимого, то
    {
        printf("Overflow.\n");      //выводим сообщение об ошибке и
        exit(ERROR);                //завершаем работу программы.
    }

    //Массив хранящий матрицу в виде которой хранится граф.
    int* adjacency_matrix;

    //Выделяем память под массив.
    adjacency_matrix = (int*)malloc(sizeof(int) * nodes_count * nodes_count);

    GetGraphFromConsole(adjacency_matrix, nodes_count);//Записываем граф в матрицу.

    CreateGraphFile(GRAPH_FILE_NAME, adjacency_matrix, nodes_count, nodes_count);//Записываем граф в файл. Так как ни накой узел пока не удалён, указываем номер удалённого узла за пределом графа.

    printf("The graph was wrote to %s\n", GRAPH_FILE_NAME);//Выводим сообщение об успешной записи графа в текстовый файл.

    ConvertGraphToPNG(GRAPH_FILE_NAME, GRAPH_IMAGE_FILE_NAME);//Создаём изображене графа.

    printf("Image was wrote to %s.\n", GRAPH_IMAGE_FILE_NAME);//Выводим сообщение об успешном создание изображение графа.

    //Номер удаляемого узла.
    int deleting_node;
    printf("\nSet number of deleting node. The graph has nodes from 0 to %i.\n", nodes_count - 1);//Сообщяем пользователю о том, что хотим считать номер удаляемого узла.
    scanf("%i", &deleting_node);//Считываем номер удаляемого узла.

    if (deleting_node < 0 || deleting_node >= nodes_count)//Если номер удаляемого узла меньше 0-я или больше либо равен количества узлов в графе. Как следствие граф не содержит данный узел.
    {
        printf("Graph has not got this node.\n");//Сообщяем пользователю, граф не имеет данного узла.
    }
    else
    {
        DeleteNode(&adjacency_matrix, &nodes_count, deleting_node);//Удаляем указанный узел из графа.
    }
    for (int x = 0; x < nodes_count; x++)
    {
        for (int y = 0; y < nodes_count; y++)
        {
            printf(" %i", adjacency_matrix[GetId(nodes_count, x, y)]);
        }
        printf("\n");
    }
    //Массив в который в последствие будет записан список узлов, отсортированных по убыванию степени.
    int* nodes_list = (int*)malloc(sizeof(int) * nodes_count);
    GetSortedNodesList(nodes_list, adjacency_matrix, nodes_count);//Заполняем массив списком узлов, отсортированных по убыванию степени.
    printf("Sorted nodes:\n");//Сообщаем пользователю, что ниже будет выведен отсортированный спсок.

    //Выводим список узлов.
    for (int node_id = 0; node_id < nodes_count; node_id++)//Проходимся по всему массиву узлов.
    {
        if (nodes_list[node_id] >= deleting_node)//Если номер узла больше либо равен номеру удалёного узла. Что означает, что после удаления узла, номер текущего узла уменьшился на единицу.
        {
            printf("Node: %i\tDegree: %i\n", nodes_list[node_id] + 1, GetNodeDegree(adjacency_matrix, nodes_count, nodes_list[node_id]));//Выводим увеличенный на единицу номер узла и его степень.
        }
        else//иначе
        {
            printf("Node: %i\tDegree: %i\n", nodes_list[node_id], GetNodeDegree(adjacency_matrix, nodes_count, nodes_list[node_id]));//Выводим номер узла и его степень.
        }
    }

    CreateGraphFile(NEW_GRAPH_FILE_NAME, adjacency_matrix, nodes_count, deleting_node);//Записываем граф в файл указывая номер удалённого узла (для правильного отображения номеров графа).

    printf("\nThe graph was wrote to %s\n", NEW_GRAPH_FILE_NAME);//Выводим сообщение об успешной записи графа в текстовый файл.

    ConvertGraphToPNG(NEW_GRAPH_FILE_NAME, NEW_GRAPH_IMAGE_FILE_NAME);//Создаём изображене графа.

    printf("Image was wrote to %s.\n", NEW_GRAPH_IMAGE_FILE_NAME);//Выводим сообщение об успешном создание изображение графа.

    free(nodes_list);
    free(adjacency_matrix);

    exit(SUCCSESS);//Завершаем работы программы.
}

void CreateGraphFile(const char* file_name, int* adjacency_matrix, int matrix_size, int deleting_node)
{
    //Указатель на файл, в который будет записываться граф. 
    FILE* f;
    f = fopen(file_name, "w+");//Открываем файл на запись, предварительно удалив его содержимое.

    fprintf(f, "graph G{\n");//Объявляем граф в нутри файла.

    for (int node = 0; node < matrix_size; node++)//перебираем все узлы графа
    {
        if (node >= deleting_node)
        {
            fprintf(f, "%i;\n", node+1);
        }
        else
        {
            fprintf(f, "%i;\n", node);
        }
        //Записываем узел в файл.
    }

    //Номер первого найденного в матрице узла имеющего ребро.
    int first_node;
    //Флаг, показывающий, найден ли прошлый узел.
    int first_node_found;

    for (int row = 0; row < matrix_size; row++)//Перебираем все строки в матрице.
    {
        for (int column = row; column < matrix_size; column++)//Перебираем все элементы на данной строке находящиеся после главной диагонали включительно.
        {
            int edges = adjacency_matrix[GetId(matrix_size, row, column)];//количество рёбер (или петель) между двумя узлами
            for (int edge = 0; edge < edges; edge++)//Перебираем все рёбра.
            {
                //номер узла который будет выведен
                int last_row = row;
                //номер узла, который будет выведен
                int last_column = column;
                if (last_row >= deleting_node)//Если номер узла больше либо равен номеру удалёного узла. Что означает, что после удаления узла, номер текущего узла уменьшился на единицу.
                {
                    last_row++;//Увеличиваем номер узла на единицу
                }
                if (last_column >= deleting_node)//Если номер узла больше либо равен номеру удалёного узла. Что означает, что после удаления узла, номер текущего узла уменьшился на единицу.
                {
                    last_column++;//Увеличиваем номер узла на единицу
                }
                fprintf(f, "%i -- %i;\n", last_row, last_column);//Записываем ребро в файл.
            }
        }
    }

    fprintf(f, "}\n");//Завершаем запись графа.
    fclose(f);//Закрываем файл в который записывали граф.

    return;//Завершаем выполнение функции.
}

void DeleteNode(int** adjacency_matrix_p, int* matrix_size_p, int node_number)
{
    //Размер входящей матрицы.
    int matrix_size;
    matrix_size = *matrix_size_p;//Получаем значение размера матрицы из указателя на эиу переменную (разыменовываем указатель).
    //Указатель на начальную матрицу.
    int* adjacency_matrix;
    adjacency_matrix = *adjacency_matrix_p;//Получаем указатель на натрицу из указателя н указатель (разыменовываем указатель на указатель)
    //Получаем значение размера матрицы из указателя на эиу переменную (разыменовываем указатель).

    if (node_number < 0 || node_number >= matrix_size)//Если граф не содержит удаляемого узла.
    {
        return;//Завершаем работу функции ни как не меняя граф.
    }

    //матрица нового графа
    int* new_matrix;
    new_matrix = (int*)malloc(sizeof(int) * (matrix_size - 1) * (matrix_size - 1));//Выделяем память для новой матрицы с учётом того что она будет содержать на одну вершину меньше.


    //Далее в новую матрицу будет переписана старая за исключением столбца и строки, номера которых равны номеру удаляемой вершины.


    for (int row = 0; row < node_number; row++)//Перебираем все строки до строки, номер которой равен номеру удаляемого узла.
    {
        for (int column = 0; column < node_number; column++)//Перебираем все столбцы до столбца, номер которого равен номеру удаляемого узла.
        {
            new_matrix[GetId(matrix_size - 1, row, column)] = adjacency_matrix[GetId(matrix_size, row, column)];//Переносим значения без сдвига.
        }
        for (int column = node_number + 1; column < matrix_size; column++)//Перебираем все столбцы до столбца номер которого следует после номера удаляемого узла и до конца.
        {
            new_matrix[GetId(matrix_size - 1, row, column - 1)] = adjacency_matrix[GetId(matrix_size, row, column)];//Переносим значения со сдвигом вверх.
        }
    }
    for (int row = node_number + 1; row < matrix_size; row++)//Перебираем все строки до строки номер которой следует после номера удаляемого узла и до конца.
    {
        for (int column = 0; column < node_number; column++)//Перебираем все столбцы до столбца, номер которого равен номеру удаляемого узла.
        {
            new_matrix[GetId(matrix_size - 1, row - 1, column)] = adjacency_matrix[GetId(matrix_size, row, column)];//Переносим значения со сдвигом влево.
        }
        for (int column = node_number + 1; column < matrix_size; column++)//Перебираем все столбцы до столбца номер которого следует после номера удаляемого узла и до конца.
        {
            new_matrix[GetId(matrix_size - 1, row - 1, column - 1)] = adjacency_matrix[GetId(matrix_size, row, column)];//Переносим значения со сдвигом вверх и влево.
        }
    }

    free(adjacency_matrix);//Очищаем память от старой матрицы (она больше не нужна).

    *adjacency_matrix_p = new_matrix;//Теперь входной указатель ссылается на новую матрицу (которая представленна ввиде массива (который хранится ввиде указателя)). Соответственно теперь указатель ссылается на новый указатель.
    matrix_size--;//Размер новой матрицы на единицу меньше размера предыдущей. Поэтому уменьшаем переменную размера матрицы
    *matrix_size_p = matrix_size;//Обновляем значение хранящееся по указателю.
    return;//Завершаем выполнение функции.
}

void GetGraphFromConsole(int* incidence_matrix, int nodes_count)
{
    for (int row = 0; row < nodes_count; row++)//Перебираем все строки в матрице.
    {
        printf("Row: %3i:  ", row);//Выводим номер текущей строки матрицы.
        for (int column = 0; column < nodes_count; column++)//Перебираем все элементы матрицы на текущей строке.
        {
            //Значение элемента матрицы.
            int value;
            scanf("%i", &value);//Потучаем значение элемента матрицы.
            if (value < 0)                                      //Если значени элемента меньше нуля, то
            {
                printf("Count of nodes is less then zero.\n");  //выводим сообщение об ошибке и
                exit(ERROR);                                    //завершаем работу программы.
            }
            incidence_matrix[GetId(nodes_count, row, column)] = value;//Записываем считанное значени в матрицу, на позицию [row, colunm]
        }
    }
    return;//Завершаем работу функции.
}

int GetId(int matrix_size, int row_number, int column_number)
{
    /*
    Все элементы матрицы пронуерованны построчно.
    Пример:
             с т о л б ц ы
             0  1  2  3  4  5

     с  0    0  1  2  3  4  5
     т  1    6  7  8  9 10 11
     р  2   12 13 14 15 16 17
     о  3   18 19 20 21 22 23
     к  4   24 25 26 27 28 29
     и  5   30 31 32 33 34 35
    */
    return row_number * matrix_size + column_number;
}

void ConvertGraphToPNG(const char* graph_file_name, const char* image_file_name)
{
    char* command;//Указатель на троку с командой.
    command = (char*)malloc(sizeof(char) * COMMAND_MAX_LENGTH);//выделяем память для строки в которую будет записанна команда для graphviz

    /*
    dot - команда
    -Tpng - аргумент (в данном случае можно назвать ключём) для команды. Обозначает, что выходной файл должен быть в формате png.
    GRAPH_FILE_NAME - название файла в котором граф записан на языке DOT
    -o - аргумент (в данном случае можно назвать ключём) для команды. Обозначает, что за ним будет следовать арнумент в котором будет указанно название файла в который будет зписанно отображение графа.
    GRAPH_IMAGE_FILE_NAME - название файла в котором записанно отображение графа
    */
    sprintf(command, "dot -Tpng %s -o %s", graph_file_name, image_file_name);//перчатаем команду (вывод происходит также, как в консоль, НО в строку)

    system(command);//Выполняем команду.

    free(command);//Очищаем память от строки.

    return;//Завершаем работу функции
}

int GetNodeDegree(const int* incidence_matrix, int matrix_size, int node_number)
{
    //Степень вешины.
    int degree;
    degree = incidence_matrix[GetId(matrix_size, node_number, node_number)];//Если узел имеет петлю, то он имеет два соединение одно из них учитывается сейчас, другое в цикле ниже..
    for (int column = 0; column < matrix_size; column++)//Переюираем все элементы в матрице инцедентности на указанной строке. (номер строки соответствует номеру узла)
    {
        degree += incidence_matrix[GetId(matrix_size, node_number, column)];//Увеличиваем значение степени на кол-во рёбер, соединённых с текущим узлом. (кол-во рёбер может равняться нулю)
    }
    return degree;//Возвращяем значение степени.
}

void GetSortedNodesList(int* nodes_list, const int* incidence_matrix, int nodes_count)
{
    //Массив хранящий список степеней узлов.
    int* degrees_list;
    degrees_list = (int*)malloc(sizeof(int) * nodes_count);//Выделяем память для массива.

    //Заолняем массив хнанящий номера узлов и массив хранящий их степени. Причём так, чтобы степень узла соответствовала его номеру.

    for (int node = 0; node < nodes_count; node++)//Перебираем все узлы.
    {
        degrees_list[node] = 0;//Устанавливаем степень текушего узла в 0.
            nodes_list[node] = node;//Заолняем номера по порядку.

        for (int column = 0; column < nodes_count; column++)//Проходимся по всем элементам строки, номер которой соответствует номеру текущего узла, в матрице.
        {
            degrees_list[node] += incidence_matrix[GetId(nodes_count, node, column)];//Увеличиваем степель узла на кол-во его соединений с другими узлами, включая соединение с самим собой. (кол-во соединений может быть равно 0)
        }
        degrees_list[node] += incidence_matrix[GetId(nodes_count, node, node)];//Учтём, что одна петля увеличивает степень не на один, а на два.
    }

    //Обычная пузырьковая сортировка.

    for (int node = 1; node < nodes_count; node++)//Перебираем все узлы кроме первого.
    {
        if (degrees_list[node - 1] < degrees_list[node])//Если предыдущий узел меньше текущего.
        {
            //переменная хранящая промежуточные значения
            int buff;

            buff = degrees_list[node - 1];              //--|
            degrees_list[node - 1] = degrees_list[node];//  } Меняем местами значения предыдущего и текущего элементов массива хранящего степени узлов.
            degrees_list[node] = buff;                  //--|

            buff = nodes_list[node - 1];                //--|
            nodes_list[node - 1] = nodes_list[node];    //  } Меняем местами значения предыдущего и текущего элементов массива хранящего номера узлов, чтобы соответствие между номером узла и его степенью продолжало выполняться.
            nodes_list[node] = buff;                    //--|

            node = 0;//Начинаем перебор с начана (значение увеличится на единицу после завершения текущей итерации цикла)
        }
    }

    return;//Завершаем работу функции.
}
