using System.Collections;
using System.Collections.Generic;
using UnityEngine;


//管理所有的UI界面
public class UIManager : MonoBehaviour {

    #region 单例
    static UIManager _instance;
    public static UIManager Instance
    {
       get
       {
            return _instance;
       }
    }
    #endregion

    //管理所有被激活过的UI界面
    Stack<UIBase> UIStack = new Stack<UIBase>();
    //保存所有进栈的UI界面
    Dictionary<string, UIBase> currentUI = new Dictionary<string, UIBase>();
    //保存Asset目录下的所有界面预制体
    public Dictionary<string, GameObject> UIObject = new Dictionary<string, GameObject>();
    
    private void Awake()
    {
        //单例实例化
        _instance = this;
        //加载所有的UI预制体
        LoadUIPrefabWithName(ConstData.UIStart);
        LoadUIPrefabWithName(ConstData.UIGame);
        LoadUIPrefabWithName(ConstData.UIOption);
        LoadUIPrefabWithName(ConstData.UIAbout);
    }
    #region 根据预制体的名字去资源路径中加载该预制
    void LoadUIPrefabWithName(string UIName)
    {
        //拼接路径
        string path = ConstData.ResourecUIDir + "/" + UIName;
        //根据路径加载预制体
        GameObject obj = Resources.Load<GameObject>(path);
        Debug.Log(obj);
        //将加载好的预制体存入集合
        if (obj)
        {
            UIObject.Add(UIName,obj);
        }
    }
    #endregion

    #region 界面入栈
    public void PushUItoStack(string uiname)
    {
        if (UIStack.Count > 0)
        {
            //返回栈顶的界面,但是不移除
            UIBase old_Pop = UIStack.Peek();
            old_Pop.OnPausing();
        }
        //创建页面
        UIBase new_Pop = GetUI(uiname);
        new_Pop.OnEntering();
        //展示这个界面
        UIStack.Push(new_Pop);
    }
    #endregion
    
    #region 创建界面（实例化界面）
    UIBase GetUI(string uiname)
    {
        //当缓存中没有存在这个ui的时候才去创建
        foreach (var name in currentUI.Keys)
        {
            //说明你要创建的UI缓存中已经存在了
            if (name == uiname)
            {
                return currentUI[uiname];
            }
        }
        //如果能来到这里说明缓存中并没有你要实例化的那个界面
        //去之前加载预制体的那个集合中去取
        GameObject uiprefabs = UIObject[uiname];
        //生成这个预制体
        GameObject objUI = Instantiate(uiprefabs);
        //为了防止生成出来的实例名字会带有clone的字样，做统一的处理
        objUI.name = uiname;
        UIBase uibase = objUI.GetComponent<UIBase>();
        //加入缓存
        currentUI.Add(uiname, uibase);
        return uibase;
    }
    #endregion

    #region 出栈
    public void PopUI()
    {
        if (UIStack.Count == 0)
        {
            return;
        }
        UIBase old_Pop = UIStack.Pop();
        old_Pop.OnExiting();
        if (UIStack.Count > 0)
        {
            UIBase newPop = UIStack.Peek();
            newPop.OnEntering();
        }
    }
    #endregion
