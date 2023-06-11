# Unity对话系统（的尝试）

先做个可以互动的NPC，添加Collider2D，设置为trigger，调整一下大小。

![image-20230611170614312](https://114514f91.oss-cn-guangzhou.aliyuncs.com/img/202306111706343.png)

然后做一个UI界面，我们需要对话的背景图，角色的名字，以及具体的对话内容。

![image-20230611170436825](https://114514f91.oss-cn-guangzhou.aliyuncs.com/img/202306111704095.png)

添加一个管理对话的工具，起名为DialogueManager，**单例**。

```c#
using TMPro;
using UnityEngine;

public class DialogueManager : Singleton<DialogueManager>
{
    public GameObject dialogueArea;
    public TextMeshProUGUI dialogueText, nameText;

    [TextArea(1, 3)]//编辑对话文字的的显示区域
    public string[] dialogueLines;
    public int currentLine;


    private void Update()
    {
        if (dialogueArea.activeInHierarchy)
        {
            if (Input.GetKeyDown(KeyCode.Space))
            {
                currentLine++;
                if (currentLine < dialogueLines.Length)
                { dialogueText.text = dialogueLines[currentLine]; }
                else
                { 
                    dialogueArea.SetActive(false); 
                    PlayerControler.Instance.isTalking = false;
                }
            }
        }
    }
    public void ShowDialogue(string[] dialogue) 
    {
        Debug.Log("调用成功！");
        dialogueLines = dialogue;
        currentLine = 0;
        dialogueText.text = dialogueLines[currentLine];
        dialogueArea.SetActive(true);
        PlayerControler.Instance.isTalking  = true;
    }
}
```

这里我们用String的数组来存储对话内容，按下空格继续对话，并且可以看到，Player脚本也是一个单例（玩家操控的角色肯定只有一个啊）。

用了单例，这样就可以在NPC对话的互动脚本里面直接调用**ShowDialogue**方法

```C#
using UnityEngine;

public class Talkable : MonoBehaviour,Iinteractable
{
    [TextArea(1, 3)]
    public string[] dialogueLines;

    public void TriggerAction()
    {
        if (DialogueManager.Instance.dialogueArea.activeInHierarchy == false)
        DialogueManager.Instance.ShowDialogue(dialogueLines);
        return ;
    }
}
```

这里让Talkable脚本继承了Iinteractable接口，接口中只包含**TriggerAction**方法

接着我们写玩家的互动脚本，让玩家可以和NPC交互

```C#
    private void OnConfirm( ) 
    {
        if (canPress && Input.GetKeyDown(KeyCode.E)) 
        {
            Debug.Log("狠狠滴调用");
            other.TriggerAction();
        }
    }

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Interactive")) 
        { canPress = true; this.other = other.GetComponent<Iinteractable>(); }   
    }

    private void OnTriggerExit2D(Collider2D other)
    {
        if (other.CompareTag("Interactive"))
        { canPress = false;}
    }
```

玩家的触发器进入可互动物体的时候，获取它的接口，然后按E键进行互动，调用接口中的**TriggerAction**

方法