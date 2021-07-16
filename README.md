Visão Geral
--------
Este é um workshop destinado a começar a criar experiência prática com o AWS CloudFormation.

O workshop propõe um modelo inicial do CloudFormation que descreve um bucket do Amazon Simple Storage Service (S3) e, em seguida, orienta o usuário para criar uma stack do CloudFormation com esse modelo: a stack criará o bucket para o usuário.

Em seguida, o workshop orientará o usuário sobre a adição interativa de valores de configuração ao modelo e para aplicar alterações à stack.

Antes de começar, certifique-se de passar pelos seguintes pré-requisitos:

- Você precisará de uma conta da AWS para executar a demonstração do workshop

- Familiarize-se com [AWS CloudFormation](https://aws.amazon.com/cloudformation/)

- Familiarize-se com o [Guia de Usuário do CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html) e mantenha-o à mão como referência

- Familiarize-se com o JSON e YAML [CloudFormation template formats](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-formats.html)

- Familiarize-se com a estrutura dos templates [Template Anatomy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-anatomy.html)

- Familiarize-se com o [Template Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-reference.html) e mantenha-o à mão em seus favoritos, pois você precisará dele ao editar seus modelos


Vamos começar! Siga as etapas abaixo:

1. Você precisará de uma cópia de trabalho do modelo inicial do CloudFormation: copie `example-s3-bucket-initial.template` para `example-s3-bucket.template`.  Por favor, note que o `example-s3-bucket.template` que você criará está listado no arquivo `.gitignore`, que deve estar no mesmo diretório do modelo inicial: a cópia de trabalho do template para este workshop será então excluída do escopo `git`.

2. A cópia de trabalho do modelo de exemplo inicial, que você fez na etapa anterior, só deve ter um recurso [S3 bucket](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html) descrito com código, e nenhuma propriedade para o bucket é especificado ainda no modelo de exemplo. Dê uma olhada no modelo e familiarize-se com sua estrutura e conteúdo: use o [CloudFormation template formats](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-formats.html) e o [template anatomy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-anatomy.html) páginas na documentação da AWS como referência. Quando você estiver pronto, vá para a próxima etapa.

3. [Faça login no CloudFormation Console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-login.html).
   Este workshop usa o Console para criar e atualizar stacks: para um em vez disso, você pode usar o [AWS Command Line Interface (CLI)](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-cli.html) ou o [AWS SDK](https://aws.amazon.com/getting-started/tools-sdks/) de sua escolha.

4. [Crie uma stack através do Console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html).
   Quando [escolhendo um template](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-console-create-stack-template.html),
   você pode escolher entre a opção `Amazon S3 URL` ou o `Upload um arquivo de Template`.  Primeiro, familiarize-se com as duas opções e entenda onde e quando você poderia usar um contra o outro. Para esta demonstração do workshop, você estará usando o último. Em seguida, inicie o processo de criação de stacks: escolha carregar sua cópia de trabalho do o modelo e, quando solicitado a especificar um nome de stack, insira `my-example-s3-bucket`.  Siga as instruções no console e depois opte por criar a stack quando estiver pronto.

1. No Console do CloudFormation, aguarde até que a stack atinja o `CREATE_COMPLETE` status: the `Events` painel para a stack que você está criando mostra etapas de criação de recursos (neste caso, seu Bucket S3)

2. Agora você começará a adicionar propriedades ao seu bucket. Você vai querer o nome do bucket para seguir este padrão:
   `my-example-bucket-AWS_REGION_NAME-AWS_ACCOUNT_ID-ENVIRONMENT_NAME`.
   As duas primeiras partes do nome podem ser derivadas automaticamente usando o `AWS::Region` and `AWS::AccountId` [Pseudo
   Parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/pseudo-parameter-reference.html).
   A última parte, o nome do ambiente, será um parâmetro de entrada: dessa forma, você pode reutilizar o mesmo modelo para criar previsivelmente e mantenha um bucket do S3 com as mesmas configurações em todos os seus ambientes (como Desenvolvimento, Garantia de Qualidade, Produção). Vamos começar adicionando, à sua cópia de trabalho do modelo, um seção principal chamada [Parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html), onde você especifica três valores predefinidos, `dev`, `qa` and `prod`,
   and let's set `dev` as the default value (you can add it after the
   `Description` section: for more information, see [Template Anatomy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-anatomy.html)
   in the AWS documentation:

       Parameters:
         EnvironmentName:
           AllowedValues:
             - dev
             - qa
             - prod
           Default: dev
           Description: Specify a value for the environment name.
           Type: String

3. Em seguida, vamos em frente e localizar o bloco nos `Resources` seção que começa com `S3Bucket`: este é o ID lógico do seu bucket. Um ID lógico é usado para fazer referência a recursos em um modelo: para obter mais informações, consulte [Resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resources-section-structure.html) na documentação da AWS. Você precisará adicionar a seção `Properties` e especificar a [propriedade para o nome do bucket](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html#cfn-s3-bucket-name). Para implementar o padrão para o nome do bucket mencionado na etapa anterior, você usará [Fn::Sub](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-sub.html),
   que é uma [função intrinseca](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resources-section-structure.html) você pode usar referência a um elemento, como um parâmetro (`EnvironmentName`) e para compor uma string ao mesmo tempo.
   Por favor, note que você também pode usar o [Ref](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html)
   função intrínseca para referenciar, por exemplo, um parâmetro no mesmo modelo e, neste caso, você está usando `Fn::Sub` para também compor uma string; para obter mais informações, consulte [Intrinsic Function Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html.
   Atualize sua cópia de trabalho do modelo, para o `S3Bucket` bloco de recursos para ter a seguinte aparência (observe: o exemplo abaixo usa o `! Sub` forma curta em oposição a `Fn። Sub` - veja uma página de documentação relevante para obter mais detalhes sobre isso):

         S3Bucket:
           Type: AWS::S3::Bucket
           Properties:
             BucketName: !Sub 'my-example-bucket-${AWS::Region}-${AWS::AccountId}-${EnvironmentName}'

4. Selecione a stack que você criou e escolha `Atualizar`. Em seguida, opte por substitua o modelo existente, carregue a nova cópia do modelo
 e escolha `Next`. Agora você terá seu primeiro parâmetro na próxima tela! Escolha `dev` (essa deve ser a seleção padrão) e siga as etapas até chegar à última tela em que você cria a stack: localize a seção “Alterar visualização do conjunto” na parte inferior do a página e aguarde até que a seção `Alterações` seja preenchida: você deve ver uma ação `Modificar` para o ID lógico `S3Bucket` (o nome lógico do intervalo que você declarou na seção Recursos do
 o modelo) e um valor de `true` para a coluna `Replacement`. As alterações feitas (novo nome do bucket neste caso) exigem um substituição do recurso: como o intervalo existente está vazio, quando você optar por atualizar a stack, a operação deve concluído com sucesso (um novo bucket é criado e o anterior será excluído - você não pode excluir um bucket com objetos nele). Para obter mais informações sobre atualizações e propriedades, consulte [Update Behaviors of Stack Resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-update-behaviors.html) na documentação da AWS. Opte por atualizar a stack e observar alterações no painel `Events` no Console e, em seguida, quando a stack estiver no estado `UPDATE_COMPLETE`, vá para a próxima etapa.

5. Agora você configurará a criptografia padrão para o bucket do S3 por especificando o [relevant property](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html#cfn-s3-bucket-bucketencryption) para seu recurso em seu modelo. Com esta demonstração, você configure a criptografia de chaves gerenciadas pelo Amazon S3 (SSE-S3). Adicione o seguinte snippet para a seção `Propriedades` do seu bucket do S3, em seguida, use o modelo atualizado para atualizar sua stack existente conforme você antes (quando a atualização da stack estiver concluída, você pode navegar para a página S3 no Console, escolha o bucket que você criou com CloudFormation e verificações de alterações feitas e aplicadas):

             BucketEncryption:
               ServerSideEncryptionConfiguration:
                 - ServerSideEncryptionByDefault:
                     SSEAlgorithm: AES256

6.  Nesta etapa, você habilitará o [property for the versioning configuration](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html#cfn-s3-bucket-versioning).
    Experimente isso sozinho desta vez, é fácil! Você precisará especificar duas linhas:

      - a primeira linha é [nome da propriedade relevante](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html#cfn-s3-bucket-versioning),

      - para a segunda linha, você precisará olhar [a propriedade relevante para configuração de controle de versão](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket-versioningconfig.html)

    Atualize seu modelo com a nova configuração e atualize a stack quando estiver pronto!

7.  Em seguida, vamos configurar a propriedade [para a configuração do ciclo de vida do objeto](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html#cfn-s3-bucket-lifecycleconfig).
    Ao configurar o ciclo de vida de objetos no bucket do S3, você configura um ou mais `LifecycleConfiguration` [rules](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket-lifecycleconfig.html):
    com esta demonstração, você configurará um exemplo [rule](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket-lifecycleconfig-rule.html) como a:

      - Mover todos os objetos com controle de versão (incluindo versões não atuais) no bucket para [Amazon Glacier](https://aws.amazon.com/glacier/) adepois de 7 dias de criação, e
      - excluir todos os objetos com mais de um ano

    Vamos tentar isso: atualize seu modelo da seguinte forma e, em seguida, atualize a pilha quando estiver pronta!

              LifecycleConfiguration:
                Rules:
                  - ExpirationInDays: 365
                    Id: MoveToGlacierAfterSevenDaysAndDeleteAfterOneYear
                    NoncurrentVersionTransitions:
                      - StorageClass: GLACIER
                        TransitionInDays: 7
                    Status: Enabled
                    Transitions:
                      - StorageClass: GLACIER
                        TransitionInDays: 7

8.  Agora você seguirá em frente e adicionará [tags](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html#cfn-s3-bucket-tags) para o seu bucket: `BusinessUnitName` e `EnvironmentName`, cujo valores que você deseja que sejam configuráveis como parâmetros. Você já tem o parâmetro `EnvironmentName`: vamos começar com a criação outro parâmetro, `BusinessUnitName`, e adicione algumas entradas controles de validação para ele (para obter mais informações, consulte [Parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html) na documentação AWS):

          BusinessUnitName:
            AllowedPattern: ^[a-zA-Z0-9_-]*$
            ConstraintDescription: Please specify a valid value for the business unit.
            Description: 'Specify a value for the business unit: minimum length: 1 character,
              maximum: 32.  Allowed characters are alphanumeric characters, underscore and
              whitespace.'
            MaxLength: 32
            MinLength: 1
            Type: String

9. Vamos continuar atualizando o modelo e adicionando a propriedade `Tags` para propriedades do balde. Você usará a função intrínseca `Ref`
 desta vez para fazer referência ao `BusinessUnitName` e `EnvironmentName` parâmetros de entrada; adicione o exemplo a seguir linhas para o modelo e atualize a stack com o atualizado modelo quando estiver pronto:

              Tags:
                - Key: BusinessUnitName
                  Value: !Ref 'BusinessUnitName'
                - Key: EnvironmentName
                  Value: !Ref 'EnvironmentName'

10. Agora você adicionará um novo recurso: a [bucket policy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-policy.html), onde você adicionará uma instrução para negar conexões inseguras (ou seja, não https) para o bucket. Para obter mais informações sobre bucket e usuário políticas para o Amazon S3, consulte [Access Policy Language Overview](https://docs.aws.amazon.com/AmazonS3/latest/dev/access-policy-language-overview.html) na documentação da AWS. A política de bucket fará referência ao bucket que você criou anteriormente em dois lugares, usando:

    - `Ref` to attach the policy to the existing bucket, and

    - `Fn::Sub` as an example of composing the [Amazon Resource Name,
       (ARN)](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html)
       do bucket que, por sua vez, você concatenará ao `/*` string para completar um valor que você indica para o `Resource` bloco

    Adicione as seguintes linhas de exemplo ao seu modelo e atualize o empilhe com o modelo atualizado quando estiver pronto:

          S3BucketPolicy:
            Type: AWS::S3::BucketPolicy
            Properties:
              Bucket: !Ref 'S3Bucket'
              PolicyDocument:
                Statement:
                  - Action: s3:*
                    Condition:
                      Bool:
                        aws:SecureTransport: 'false'
                    Effect: Deny
                    Principal: '*'
                    Resource: !Sub 'arn:aws:s3:::${S3Bucket}/*'
                    Sid: DenyInsecureConnections
                Version: '2012-10-17'

11.  No final deste workshop, encerre a stack para excluir recursos (política de bucket e bucket) que você criou com ele.
