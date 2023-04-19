# PiCachu.Documentaion
## system components diagram

```plantuml 
  @startuml C4_Elements
  !include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml
  HIDE_STEREOTYPE()
  
  AddElementTag("queue", $bgColor="Teal",$borderColor="black")
  AddElementTag("exchange", $bgColor="SeaGreen",$shape=RoundedBoxShape())
  AddElementTag("neural_network", $bgColor="DarkSlateGray")
  AddElementTag("database", $bgColor="MediumSlateBlue",$borderColor="black")
  AddElementTag("storage", $bgColor="LightSlateGray",$borderColor="black")
  
  Container(UI, "UI", "", "")
  Container(API, "API", "", "")
  
  ContainerQueue(ColorServiceQueue, "Color\nService\nQueue", "the system starts\n three queues\nfor each model\n1.ColorQueue\n2.EmotionQueue\n3.ObjectQueue}", "",$tags = "queue")

  ContainerDb(S3, "S3", "", "",$tags="storage")
  ContainerDb(DB, "DB", "", "",$tags="database")
  ContainerDb(DB_tags, "DB_tags", "", "", $tags="database")
  ContainerQueue(DelayedRetryQueue, "Delayed\nRetry\nQueue", "", "",  $tags = "queue")
  Container(ColorService, "Color\nService", "the system starts\n three models\n1.ColorModel\n2.EmotionModel\n3.ObjectModel}", "",$tags="neural_network")

  Container(RequestExchange, "Request\nExchange", "", "", $tags ="exchange")
  
  Container(ModelsQueuesDLX, "Models\nQueuesDLX", "", "",$tags ="exchange")
  Container(DelayedRetryQueueDLX, "Delayed\nRetry\nQueueDLX", "", "",$tags ="exchange")
   
  Container(ResultService, "Result\nService", "", "")
  ContainerQueue(ResultQueue, "Result\nQueue", "", "",$tags = "queue")
  
  ContainerQueue(AssociationsQueue, "Associations\nQueue", "", "",$tags = "queue")
  Container(AssociationsService, "Associations\nService", "", "",$tags="neural_network")
  
  Container(UniquenessService, "Uniqueness\nService", "", "")
  ContainerQueue(UniquenessQueue, "Uniqueness\nQueue", "", "",$tags = "queue")
  ContainerQueue(ResultUniquenessQueue, "Result\nUniqueness\nQueue", "", "",$tags = "queue")
  
  Rel_D(UI, API, " ", "")
  
  Rel_D(API, RequestExchange, " ", "")
  Rel_L(API, S3, "POST", "save photo")
  Rel_U(API, DB, " ", "")
  Rel_L(RequestExchange, ColorServiceQueue, "message", "photo_id")
  Rel_R(ColorService, ColorServiceQueue, "listen", "")
  Rel_U(ColorService, S3, "GET", "photo")
  
  Rel(ColorService, ResultQueue, "on success\nsend message", "photo_id;\ntype model")
  Lay_U(ResultQueue,ColorServiceQueue)

  
  Lay_U(AssociationsQueue,ColorServiceQueue)
  
  Rel_R(AssociationsService, AssociationsQueue, "listen", "")
  
  Lay_U(AssociationsService,ModelsQueuesDLX)
  Lay_U(AssociationsService,ColorService)
  Lay_U(DB_tags,RequestExchange)
  
  
  Rel_D(ColorService, ModelsQueuesDLX, "on reject", "")
  Rel_U(AssociationsService, ModelsQueuesDLX, "on reject", "")
  
  Rel_L(ModelsQueuesDLX, DelayedRetryQueue, " ", "")
  Rel_L(DelayedRetryQueue, DelayedRetryQueueDLX, " ", "")
  
  Rel(DelayedRetryQueueDLX, ColorService, " ", "")
  
  
  Lay_R(DelayedRetryQueueDLX,DelayedRetryQueue)
  Lay_R(DelayedRetryQueue,ModelsQueuesDLX)
  Lay_U(DelayedRetryQueueDLX,ColorService)
  
  Rel(DelayedRetryQueueDLX, AssociationsService, " ", "")
  Rel_U(AssociationsService, ResultQueue, "on success\nsend message", "photo_id;\ntype model")
  
  Rel_L(ResultService, ResultQueue, "listen", "")
  
  Rel_D(ResultService, AssociationsQueue, "if get all tags\nsend message", "photo_id;\ntags")
  Rel_D(ResultService, DB_tags, " ", "")
  
  Rel_R(ResultService, UniquenessQueue, "if get color\nor association\nsend message ", "photo_id\ntype_metric")
  Rel_D(UniquenessService, UniquenessQueue, "listen", "")
  
  Rel_D(UniquenessService, ResultService, "GET", "tags from DB")
  
  Rel_U(UniquenessService,ResultUniquenessQueue ,  "message", "photo_id\ntype_metric\nvolume_metric")
  
  Lay_U(UniquenessService,UniquenessQueue)
  Lay_R(RequestExchange,UniquenessService)
  
  Rel_R(API, ResultUniquenessQueue,  "listen", "")
  
  Rel_U(UniquenessService,API ,  "GET", "gallery photos ids\nby source photo id")
  
  SHOW_LEGEND(false)
  @enduml
```
