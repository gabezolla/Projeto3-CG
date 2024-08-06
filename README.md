# Round 6 - Soldiers
### Projeto 3 - Computação Gráfica

Autores: **Gabriel Zolla Juarez - RA: 11201721446**

**Pedro Henrique Batistela Lopes - RA: 11201722043**

**Link do repositório do código-fonte:** https://github.com/gabezolla/Projeto3-CG/tree/main/examples/atividade3

**Link para o vídeo demonstrativo:** https://youtu.be/ZBtz5WcItsI

Este repositório contém o código-fonte do Round-6, projeto desenvolvido para a disciplina de Computação Gráfica, ministrada pelo Prof. Bruno Dorta.

### :red_square: :small_red_triangle: :red_circle: Squid Game :red_circle: :small_red_triangle: :red_square:

Squid Game foi a série mais vista na Netflix nos últimos tempos, cuja reação do público foi extremamente positiva. A história se baseia em pessoas que, necessitando de dinheiro, aceitam um convite para um jogo peculiar, com inúmeras fases, envolvendo sobrevivência, raciocínio e sorte, em busca de um prêmio bilionário. Neste jogo, existem inúmeros soldados com a função de fiscalizar e executar os participantes que burlam regras ou quando acabam pendendo nas provas citadas. Todos eles usam máscaras que contêm três símbolos: círculo, triângulo e quadrado.

A interpretação mais comum para esses símbolos é referente à hierarquias soldados, dependendo da função dos mesmos. Aqueles cujas máscaras possuem forma de círculo são encarregados dos trabalhos braçais, como a limpeza e o descarte dos corpos. As de triângulos, por sua vez, são protetores, executores e estão sempre armados. Os de símbolo quadrado, por fim, supervisionam os outros soldados e os jogadores. Acima de todos eles está o criador do jogo, que usa uma roupa escura e uma máscara peculiar, de caráter geométrico. 

<hr style="border:1px solid gray"> </hr>

### :page_with_curl: **Relatório** :page_with_curl:

A implementação consiste na renderização de soldados da renomada série 'Round 6'. Para tal, utilizar-se-á, modelos (arquivos .obj) retirados de fontes da internet e aplicamos conceitos iniciais de renderização visto nas aulas anteriores, utilizados também no projeto 2, bem como manipulação do modelo e atributos como translation, rotation e scale, e também concepções mais avançadas, como texturização e iluminação. Os modelos, por sua vez, foram criados por seus autores baseado no pipeline gráfico do OpenGL, a partir de arranjos de vértices. 

Nos projetos anteriores, os objetos foram desenhados a partir de fragmentos de mesma cor pré-definida pela aplicação, estratégia muito mais simplificada. Para o projeto 3, Round 6 - Soldiers, abranger-se-á o modelo de iluminação. Esse modelo descreve a relação entre a luz incidente em cada ponto de uma superfície e a luz refletida em uma dada direção, que, por sua vez, varia de acordo com as propriedades do material da superfície do objeto escolhido e das quantidades e propriedades das fontes de luz incidente e refletida. Esse comportamento é descrito pela equação de renderização, que engloba as intensidades luminosas espalhadas sob todo o sólido geométrico, respeitando integralmente as leis da física (como as da óptica). No entanto, por conta do fato de que a solução dessa equação demanda grande poder computacional, pode-se simplificar de forma a considerar apenas as luzes pontuais diretas no cenário, os denominados modelos de iluminação locais, tal como o modelo de reflexão de Phong e o de Blinn-Phong, este último sendo o utilizado objetos do projeto.Na mesma linha, destaca-se o sombreamento, que consiste em diferentes intensidade de cores da imagem (claros e escuros), para gerar a percepção de profundidade e volume de um objeto 3D. Nesse caso, existem três modelos abordados em aula: flat (avalia o modelo de iluminação para cada face), Gouraud (para cada vértice) e Phong (para cada fragmento). Novamente, o último será usado no Round 6 - Soldiers.

A texturização, por sua vez, será outro conceito crucial para o projeto. Simplificadamente, consiste em mapear pontos de uma imagem tridimensional através de um mapa de textura (imagem 2D). Neste, existem os texels, que são pixels de textura, isto é, são a unidade fundamental de um mapa de textura. Para realizar o processo de texturização de forma correta, é necessário conhecer três passos base para o mesmo. 

Primeiro, o mapeamento, uma função que mapeia pontos 2D da textura para o 3D, que, por sua vez, é o objeto final, e, para isso, pode-se utilizar mapeamento planar, cilíndrico, esférico e UV-unwrap. Este último é comum em texturização de personagens em jogos, por exemplo, e será utilizado em nosso projeto. 

Em seguida, tem-se o empacotamento, que descreve o comportamento que o modelo deve tomar em pontos fora do intervalo. Para tal, tem-se três comportamentos: GL_REPEAT (repete a textura fora do intervalo), GL_MIRRORED_REPEAT (repete a textura, porém é espelhada eventualmente), e GL_CLAMP_TO_EDGE (fixa as coordenadas no intervalo (0,1), repetindo os valores das primeiras e últimas linhas/colunas da textura). 

Por fim, existe a filtragem, que lida com os problemas de pixels que correspondem a vários texels e/ou texels que podem ser mapeado a vários pixels, que, respectivamente, são as soluções denominadas de magnificação e minificação. Na magnificação, tem-se a interpolação por vizinho mais próximo, que utiliza a distância de Manhattan para pegar o valor do texel que está mais próximo da posição da amostragem, e a bilinear, a filtragem padrão do OpenGL, que consiste em realizar uma média entre os quatro texels mais próximos da posição. A minificação, por sua vez, além dos anteriores, temos o mipmapping, para evitar problemas de aliasing quando quatro texels seriam mapeados para um mesmo pixel. Basicamente, o mipmapping consiste em filtrar texels a partir de mipmaps.

Seguindo as propriedades descritas anteriormente, utilizar-se-á o modelo de Blinn-Phong, empacotamento GL_REPEAT, e filtragem (tanto magnificação quanto minificação) linear. Respectivamente, a partir dos seguintes blocos de código:

* Blinn-Phong
```
vec4 BlinnPhong(vec3 N, vec3 L, vec3 V, vec2 texCoord) {
  N = normalize(N);
  L = normalize(L);

  // Compute lambertian term
  float lambertian = max(dot(N, L), 0.0);

  // Compute specular term
  float specular = 0.0;
  if (lambertian > 0.0) {
    V = normalize(V);
    vec3 H = normalize(L + V);
    float angle = max(dot(H, N), 0.0);
    specular = pow(angle, shininess);
  }

  vec4 map_Kd = texture(diffuseTex, texCoord);
  vec4 map_Ka = map_Kd;

  vec4 diffuseColor = map_Kd * Kd * Id * lambertian;
  vec4 specularColor = Ks * Is * specular;
  vec4 ambientColor = map_Ka * Ka * Ia;

  return ambientColor + diffuseColor + specularColor;
}
```

* Empacotamento
```
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
```

* Filtragem
```
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

Vale ressaltar que o arquivo .obj que foi utilizado no projedo acompanha um arquivo .mtl que contém a descrição das propriedades dos materiais dos objetos, bem como a localização dos pontos de mapeamento da textura.

Além disso, destacar-se-á o uso de mapeamento cúbico (do inglês cubemap), que se trata de um conjunto de seis imagens que juntas formam um cubo que forma o ambiente. Esse mapa de texturas é carregado em:

```
void Model::loadCubeTexture(const std::string& path) {
  if (!std::filesystem::exists(path)) return;

  glDeleteTextures(1, &m_cubeTexture);
  m_cubeTexture = abcg::opengl::loadCubemap(
      {path + "px.png", path + "nx.png", path + "py.png",
       path + "ny.png", path + "pz.png", path + "nz.png"});
}
```

Por fim, fez-se o uso também de uma biblioteca externa SDL para permitir a alocação de buffers para transmitir sons. No caso, optamos pela música característica da série Squid Game, que gera uma ambientalização ao aplicativo. Essa funcionalidade foi desenvolvida a partir da função abaixo:

```
void OpenGLWindow::initializeSound(std::string path){
  // clean up previous sounds
  SDL_CloseAudioDevice(m_deviceId);
  SDL_FreeWAV(m_wavBuffer);

  SDL_AudioSpec wavSpec;
  Uint32 wavLength;

  if (SDL_LoadWAV(path.c_str(), &wavSpec, &m_wavBuffer, &wavLength) == nullptr) {
    throw abcg::Exception{abcg::Exception::Runtime(
        fmt::format("Failed to load sound {} ({})", path, SDL_GetError()))};
  }

  m_deviceId = SDL_OpenAudioDevice(nullptr, 0, &wavSpec, nullptr, 0);

  if (SDL_QueueAudio(m_deviceId, m_wavBuffer, wavLength) < 0) {
    throw abcg::Exception{abcg::Exception::Runtime(
        fmt::format("Failed to play sound {} ({})", path, SDL_GetError()))};
  }

  SDL_PauseAudioDevice(m_deviceId, 0);
}
```


<hr style="border:1px solid gray"> </hr>

### :joystick: **Controles** :joystick:

Os controles são simples: o usuário pode movimentar o soldado com o mouse, bem como selecionar o seu soldado no menu do canto inferior direito da tela. 

<hr style="border:1px solid gray"> </hr>
