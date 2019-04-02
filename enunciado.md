

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

    rosrun p1_ros p1.py

# Questões

## 1.

Modifique o código para que seu robô sempre ande para a frente quando enxergar pelo menos um pássaro

## 2. 

Modifique o código do item anterior para que seu robô sempre ande para trás quando enxergar um círculo

## 3.

Identifique quando as questões **(1)** e **(2)** acontecerem ao mesmo tempo. Decida o que o robô deverá fazer

## 4. 

Explique como você faria se só interessasse a você andar para trás quando o círculo fosse azul (RGB 255,0,0) ou mais escuro.

## 5. 

Você descobriu que seu robô só tem bateria para andar mais 50 metros.

Pesquise sobre o tópico `/odom` e diga como você faria para que seu robô só andasse 50 metros ou menos. Você precisa dizer em termos de código o que precisaria ser feito, sem fazer efetivamente

## 6. 

Responda o questionário sobre o simulado enviado por e-mail


