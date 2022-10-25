# Portfolio

#### [개발 기간] 22.05 ~ 22.07
#### [개발 인원] 1인
#### [구현 내용]
+ [데이터 베이스](#데이터-베이스)
+ [인벤토리 아이템 추가](#인벤토리-아이템-추가)
+ [슬롯 내의 아이템 정보](#슬롯-내의-아이템-정보)
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

+ #### **_아이템 버튼 클릭_**

```c#
    public void _On_AddItemBtnClick()
    {
        GameObject current_click_btn = EventSystem.current.currentSelectedGameObject;     // 현재 클릭한 게임 오브젝트
        Item current_click_item = current_click_btn.GetComponent<Item_Scriptable>().item; // 클릭한 아이템 정보

        Inventory.insert_item_to_inventory(current_click_item);                           // 인벤토리에 클릭한 아이템 추가
    }
```

+ #### **_인벤토리 아이템 추가 로직_**
    + _1. 빈 슬롯_
    + _2. 추가할 아이템과 같은 아이템 이고, 슬롯이 가득 차 있지 않음_
    + _3. 추가할 아이템의 남은 개수가 1개 이상_
```c#
    public static void insert_item_to_inventory(Item insert_item)
    {
        IEnumerator<Slot> enumerator = slot_list.GetEnumerator();
        int insert_item_count = insert_item.is_stackable ? MaxItemStack.stackable : MaxItemStack.non_stackable;

        while (enumerator.MoveNext())
        {
            Slot current_slot = enumerator.Current;                         
            Item current_item = current_slot.item_info.get_top_item_info(); 

            if (null == current_item)
            {
                for (int i = 0; i < insert_item_count; i++)
                    current_slot.item_info.item_stack.Push(insert_item);

                insert_item_count = 0;
            }
            else if(insert_item == current_item)
            {
                while (false == current_slot.item_info.is_item_stack_full() && insert_item_count > 0)
                {
                    current_slot.item_info.item_stack.Push(insert_item);
                    --insert_item_count;
                }
            }
            current_slot.item_info.update_UI();       
            if (0 == insert_item_count) break;        
        }
    }
```

## 슬롯 내의 아이템 정보

+ #### **_아이템 정보 클래스 사용처_**
    + _1. 인벤토리, 작업대 슬롯_
    + _2. 완성 아이템 슬롯_
    + _3. 드래그 아이템_
    
```c#
public class ItemInfo
{
    public Stack<Item> item_stack;
    public Image item_image;                  
    public Text item_quantity_text;

    public ItemInfo() { // ... 변수 초기화 }

    // 아이템 변경 내용 있을 때마다 호출하여 UI 업데이트
    public void update_UI()
    {
        // 아이템 스택이 비었을 때
        if (true == is_item_stack_empty())
        {
            item_image.sprite = null;
            item_image.enabled = false;

            item_quantity_text.text = item_stack.Count.ToString();
            item_quantity_text.enabled = false;
            return;
        }

        Item current_item = get_top_item_info();
        item_image.enabled = true;
        item_image.sprite = current_item.item_sprite;
        // 쌓을 수 있는 아이템, 아이템 스택이 1보다 크면 개수(Text) 표시
        item_quantity_text.enabled = current_item.is_stackable ? true : false;
        item_quantity_text.enabled = 1 == get_item_stack_quantity() ? false : true;
        item_quantity_text.text = item_stack.Count.ToString();
    }

    // 현재 스택에 쌓인 아이템 개수 가져가기
    public int get_item_stack_quantity() { // ... }

    // 아이템의 최대 스택 개수 가져가기
    public int get_max_item_stack() { // ... }

    // 아이템 정보 가져가기
    public Item get_top_item_info() { // ... }

    // 아이템 이름 가져가기
    public string get_top_item_name() { // ... }

    // 아이템 스택이 가득 찼는가?
    public bool is_item_stack_full() { // ... }

    // 아이템 스택이 비어 있는가?
    public bool is_item_stack_empty() { // ... }
}
```

## 슬롯 클릭 처리

+ #### **_인벤토리, 작업대 슬롯 클릭 IPointerClickHandler 사용_**
    + _슬롯 클릭 상황 별 처리_
        + _Non Dragging_
            + _빈 슬롯 :: 함수 종료_
            + [_아이템 슬롯 :: 아이템 드래그_](#아이템-드래그)
        + _Dragging_
            + [_빈 슬롯 :: 아이템 드랍_](#아이템-드랍)
            + [_아이템 슬롯 :: 아이템 드랍 or 스왑_](#아이템-드랍-&-스왑)

#### **_아이템 드래그_**


#### **_아이템 드랍_**


#### **_아이템 드랍 & 스왑_**
## 제작대 아이템 조합
