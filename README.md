# Portfolio

#### [개발 기간] 22.05 ~ 22.07
#### [개발 인원] 1인
#### [구현 내용]
+ [데이터 베이스](#데이터-베이스)
+ [인벤토리 아이템 추가](#인벤토리-아이템-추가)
+ [슬롯 클릭 처리](#슬롯-클릭-처리)
+ [제작대 아이템 조합](#제작대-아이템-조합)

## 데이터 베이스
### 아이템 :: ScriptableObject

ScriptableObject 생성을 위한 클래스

```c#
[CreateAssetMenu(fileName = "New Item", menuName = "New Item/Item")]
public class Item : ScriptableObject
{
    public string item_name;                      // 아이템 이름
    public EItemType item_type;                   // 아이템 타입 (장비, 소비)
    public Sprite item_sprite;                    // 아이템 스프라이트
    public bool is_stackable;                     // 아이템을 쌓을 수 있는가?
    public bool is_material;                      // 재료 아이템인가?
}
```

<img src="/Image/ScriptableObject.PNG" height="60%" width="80%">

### 아이템 레시피 :: CSV

CSVReader 오픈 소스 사용

```c#
public class ItemRecipe
{
    private string[,] recipe;       // 아이템 레시피
    private int create_quantity;    // 아이템 생성 개수
    private int material_quantity;  // 필요한 총 재료 개수

    public ItemRecipe()
    {
        recipe = new string[3, 3];
        create_quantity = 0;
        material_quantity = 0;
    }
    
    // ...프로퍼티
}
```

<img src="/Image/RecipeCSV.png" height="60%" width="80%">

## 인벤토리 아이템 추가
## 슬롯 클릭 처리


















## 제작대 아이템 조합
