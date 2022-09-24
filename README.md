# Portfolio

#### [개발 기간] 22.05 ~ 22.07
#### [개발 인원] 1인
#### [구현 내용]
+ [데이터 베이스](#데이터-베이스)
+ [인벤토리 아이템 추가](#인벤토리-아이템-추가)
+ [슬롯 클릭 처리](#슬롯-클릭-처리)
+ [제작대 아이템 조합](#제작대-아이템-조합)

## 데이터 베이스
+ ### 아이템 :: ScriptableObject

    + *__ScriptableObject__ 생성을 위한 클래스*   

```c#
[CreateAssetMenu(fileName = "New Item", menuName = "New Item/Item")]
public class Item : ScriptableObject
{
    public string item_name;                      // 아이템 이름
    public EItemType item_type;                   // 아이템 타입
    public Sprite item_sprite;                    // 아이템 스프라이트
    public bool is_stackable;                     // 아이템을 쌓을 수 있는가?
    public bool is_material;                      // 재료 아이템인가?
}
```

<img src="/Image/ScriptableObject.PNG" height="60%" width="100%">

+ ### 아이템 레시피 :: CSV

    + _CSVReader 오픈 소스 사용_

```c#
public class ItemRecipe
{
    private string[,] recipe;       // 아이템 레시피
    private int create_quantity;    // 아이템 생성 개수
    private int material_quantity;  // 필요한 재료 개수

    public ItemRecipe()
    {
        recipe = new string[3, 3];
        create_quantity = 0;
        material_quantity = 0;
    }
    
    // ...프로퍼티
}
```

_**Recipe.csv**_

<img src="/Image/RecipeCSV.png" height="60%" width="100%">

+ ### ItemDataBase
    + _아이템, 아이템 레시피 정보 [SET, GET]_

```c#
public class ItemDataBase : Singleton_Mono<ItemDataBase>
{
    public Dictionary<string, Item> item_database = null;                      // 아이템 정보
    public List<Dictionary<string, ItemRecipe>> item_recipe_database = null;   // 아이템 레시피
    
    // ... Awake :: Initialize, Set info
    
    // ... Initialize :: database 동적 할당

    // 아이템 레시피 정보 가져가기
    public Dictionary<string, ItemRecipe> get_item_recipe_data(int material_quantity)
    {
        return item_recipe_database[material_quantity];
    }

    // 아이템 정보 저장 :: ScriptableObject
    private void set_items_info()
    {
        Item[] items_info = Resources.LoadAll<Item>("ScriptableObject/Items");      
        int item_count = items_info.Length;                                        

        for (int i = 0; i < item_count; i++)
            item_database.Add(items_info[i].item_name, items_info[i]);           
    }

    // 아이템 레시피 정보 저장 :: CSV
    private void set_items_recipe_info()
    {
        List<Dictionary<string, object>> csv_data = CSVReader.Read("CSV/Recipe");
        int recipe_count = csv_data.Count;

        for (int i = 0; i < recipe_count; i++)
        {
            ItemRecipe curr_item_recipe = new ItemRecipe();  // 현재 아이템 레시피
            string item_name = "";                           // 조합 아이템 이름

            item_name = csv_data[i]["조합 아이템"] as string;
            curr_item_recipe.CreateQuantity = int.Parse(csv_data[i]["생성 개수"].ToString());

            for (int j = 0; j < 3; j++)
            {
                for (int k = 0; k < 3; k++)
                {
                    string material_item_name = csv_data[i][$"\"{j},{k}\""] as string;
                    curr_item_recipe.Recipe[j, k] = string.IsNullOrEmpty(material_item_name) ? string.Empty : material_item_name;
                    curr_item_recipe.MaterialQuantity += string.IsNullOrEmpty(material_item_name) ? 0 : 1;                  
                }
            }

            item_recipe_database[curr_item_recipe.MaterialQuantity].Add(item_name, curr_item_recipe);
        }
    }
}

```

## 인벤토리 아이템 추가
<p align="center">
    <img src="/Image/Material Item Button.png" align="center" width="49%">
    <img src="/Image/Add Item to Inventory.png" align="center" width="49%">
    <figcaption align="center"></figcaption>
</p>


## 슬롯 클릭 처리


















## 제작대 아이템 조합
