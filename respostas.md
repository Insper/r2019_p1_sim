

# Robótica 2019 - prova 1 - simulado

Copie os seguintes arquivos do demo de *mobilenet* da aula03 para a pasta `~/catkin_ws/src/r2019_p1_sim/p1_ros/scripts`

    MobileNetSSD_deploy.caffemodel
    MobileNetSSD_deploy.prototxt.txt

Baixe o conteúdo do repositório para seu ~/catkin_ws/src/

    git clone https://github.com/insper/r2019_p1_sim

Em seguida:

    cd ~/catkin_ws
    catkin_make

Para executar o código-base, faça o seguinte:


    cd ~/catkin_ws/src/r2019_p1_sim/p1_ros/scripts

    rosrun p1_ros p1.py

# Questões

## 1.

Modifique o código para que seu robô sempre ande para a frente quando enxergar pelo menos um pássaro

### Resposta questão 1

A primeira medida é analisar a estrutura do código: temos `p1.py` que importa `visao_module.py`, 
que por sua vez importa `mobilenet_simples.py`. Esta última é o código de *MobileNet* da Aula 03, mas simplificado.

A linha `51` é fundamental para entendermos o resultado da *MobileNet*, com a retorno através da variável `resultados` 

```python
		centro, imagem, resultados =  visao_module.processa(cv_image)
```

Vamos adicionar um `print` para inspecionar a variável `resultados`. Vemos que esta é uma **lista de tuplas** com os objetos detectados pela mobilenet:
```python
('bird', 92.822962999343872, (445, 282), (680, 433))
('diningtable', 97.974604368209839, (27, 59), (1278, 705))
('person', 59.926551580429077, (0, -13), (468, 677))
```

Da aula de *MobileNet*, lembramos que estes elementos são, respectivamente, `categoria`, `confianća`, `ponto inicial` e `ponto final`

O commit para identificar a adicão do pássaro pode ser visto [no Github aqui](https://github.com/Insper/r2019_p1_sim/commit/8b7dfec354650fb062a18a12ec8a9e2b60ba3299) , mas está resumido abaixo.

```python
for r in resultados:
    # print(r) - print feito para documentar e entender
    # o resultado
    if r[0] == "bird":
        viu_bird = True
```

## 2. 

Modifique o código do item anterior para que seu robô sempre ande para trás quando enxergar um círculo

Vamos portar os trechos de `draw_circles_video.py` que permitirão detectarmos um círculo.

Este commit está [principalmente aqui](https://github.com/Insper/r2019_p1_sim/commit/76dca99236ebcd5ef0d98427448bc766ff8ddd7b)

```python
def auto_canny(image, sigma=0.33):
    # compute the median of the single channel pixel intensities
    v = np.median(image)

    # apply automatic Canny edge detection using the computed median
    lower = int(max(0, (1.0 - sigma) * v))
    upper = int(min(255, (1.0 + sigma) * v))
    edged = cv2.Canny(image, lower, upper)

    # return the edged image
    return edged

def find_circles(imagem_bgr):

    gray = cv2.cvtColor(imagem_bgr, cv2.COLOR_BGR2GRAY)
    # A gaussian blur to get rid of the noise in the image
    blur = cv2.GaussianBlur(gray,(5,5),0)
    #blur = gray
    # Detect the edges present in the image
    bordas = auto_canny(blur)
    bordas_color = cv2.cvtColor(bordas, cv2.COLOR_GRAY2BGR)
    circles = None

    # Not: precisamos aumentar o maxRadius - veja a documentacao
    circles=cv2.HoughCircles(bordas,cv2.HOUGH_GRADIENT,2,40,param1=50,param2=100,minRadius=5,maxRadius=500)

    if circles is not None:
        circles = np.uint16(np.around(circles))

        for i in circles[0,:]:
            print(i)
            # draw the outer circle
            # cv2.circle(img, center, radius, color[, thickness[, lineType[, shift]]])
            cv2.circle(bordas_color,(i[0],i[1]),i[2],(0,255,0),2)
            # draw the center of the circle
            cv2.circle(bordas_color,(i[0],i[1]),2,(0,0,255),3)

    cv2.imshow("circulos", bordas_color)

    if circles is not None:
        return True
    else:
        return False



```


## 3.

Identifique quando as questões **(1)** e **(2)** acontecerem ao mesmo tempo. Decida o que o robô deverá fazer

### Resposta

No código  da forma que está no gabarito (ver o while do main) o robô iria vai para a frente porque viu um pássaro, e em seguida pula a parte de ir para trás por causa do `continue`

```python
            vel = Twist(Vector3(0,0,0), Vector3(0,0,0))

            if viu_bird:
                vel = Twist(Vector3(0.4,0,0), Vector3(0,0,0))
                velocidade_saida.publish(vel)
                rospy.sleep(0.8)
                viu_bird = False
                continue

            if viu_circulo:
                vel = Twist(Vector3(-0.4,0,0), Vector3(0,0,0))
                velocidade_saida.publish(vel)
                rospy.sleep(0.8)
                viu_circulo = False
                continue

            velocidade_saida.publish(vel)
            rospy.sleep(0.1)

```

## 4. 

Explique como você faria se só interessasse a você andar para trás quando o círculo fosse azul (RGB 0,0,255) ou mais escuro.

### R.:

É possível filtrar (usando `inRange` ou o canal `R` de uma imagem RGB para selecionar primeiro os pixels azuis) obtendo uma imagem inicial

A partir desta selećão é possível usar o detector de bordas de *Canny* e a seguir partir para o `Houghcircles` 

## 5. 

Você descobriu que seu robô só tem bateria para andar mais 50 metros.

Pesquise sobre o tópico `/odom` e diga como você faria para que seu robô só andasse 50 metros ou menos. Você precisa dizer em termos de código o que precisaria ser feito, sem fazer efetivamente


Vamos analisar o tópico `/odom`

```bash
borg@ubuntu:~/catkin_ws/src/r2019_p1_sim$ rostopic info /odom
Type: nav_msgs/Odometry

Publishers: 
 * /gazebo (http://ubuntu:35619/)

Subscribers: None
```

Descobrimos que o `/odom` é do tipo `nav_msgs/Odometry`

```bash
borg@ubuntu:~/catkin_ws/src/r2019_p1_sim$ rosmsg info nav_msgs/Odometry
std_msgs/Header header
  uint32 seq
  time stamp
  string frame_id
string child_frame_id
geometry_msgs/PoseWithCovariance pose
  geometry_msgs/Pose pose
    geometry_msgs/Point position
      float64 x
      float64 y
      float64 z
    geometry_msgs/Quaternion orientation
      float64 x
      float64 y
      float64 z
      float64 w
  float64[36] covariance
geometry_msgs/TwistWithCovariance twist
  geometry_msgs/Twist twist
    geometry_msgs/Vector3 linear
      float64 x
      float64 y
      float64 z
    geometry_msgs/Vector3 angular
      float64 x
      float64 y
      float64 z
  float64[36] covariance

```

**Resposta:** Vamos precisar fazer um objeto *Subscriber* para mensagens do tipo *nav_msgs/Odometry* e um método para receber notificacões. As mensagens de odometria tem variáveis `position.x`, `position.y` e `position.z` que trarão a posicão do robô .

Basta descobrirmos a posićão atual usando a primeira notificaćão de odometria e em seguida monitorar até que a distância entre a posićão atual a e inicial seja de $50m$. Quando a distância for de $50m$ deveremos parar o robô para evitar que o mesmo fique sem bateria.


## 6. 

Responda o questionário sobre o simulado enviado por e-mail


