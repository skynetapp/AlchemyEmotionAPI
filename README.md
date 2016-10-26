#### Date: 25-10-2016
#### Description: This document aims to define the Alchemy Emotion API coding 


#### The Folder Structure is as follows:
   
   
   Root Directory | Sub Directory | Sub Directory 
------------ | ------------- | -------------
index.php | | |
Global | DBmongo(Mongo Connection),DBmysql(MySQL Connection), AlchemyAPI(Alchemy API Connection)  | 
Lib | Smarty,Common functions,AlchemyAPI | |
Modules | AlchemyEmotion | Alchemy Emotion Controller, Alchemy Emotion Action, Alchemy Emotion MySql, Alchemy Emotion View, Alchemy Emotion DB Mongo|
Views | AlchemyEmotion | header.tpl, footer.tpl(Common files), masterList.tpl,detailList.tpl|

#### Code Flows as follows:
   * To insert or get data from DB code flows.. index.php -> Controller -> Action -> MySql.
   * To view the data code flows.. index.php -> Controller -> View.
   
 
#### Step 1:
  Add the created Url and Alchemy Key in the config.php under bluemix2.0 folder.
	
**_Code:_**
	
```
	
$GLOBALS['alchemy_apiKey']='75c6700778eda02ffce454aed26b51f1faad5f54';
$GLOBALS['alchemy_url']='https://gateway-a.watsonplatform.net/calls';
	
```
	
  
#### Step 2:
  Create a module name as 'alchemyEmotion' in index.php and respective actions will be performed accordingly.
  
**_Code:_**

```
<?php
if($_REQUEST['module']=='alchemyEmotion'){
    switch ($_REQUEST['action']){
        case 'GetList':
        {
			//echo "1";exit; 
            $alchemyController = AlchemyEmotionController::getInstance();
            $alchemyController->getDataFromAlchemyDB();
            break;
        }
		 case 'GetMasterList':
        {
			//echo "1";exit; 
            $alchemyController = AlchemyEmotionController::getInstance();
            $alchemyController->getMasterListData();
            break;
        }
        case 'DetailList':
        {
            $alchemyController = AlchemyEmotionController::getInstance();
            $alchemyController->getEmotionChildDataFromMySQL($_REQUEST);
            break;
        }
      
    }
}
?>

```

#### Step 3:
   In the server normally we will be able to see existing Master list data. When we click on the update button **GetList** action will be performed from index page and **getDataFromAlchemyDB()** will be called.
   
#### Step 4:
   From index.php, **AlchemyEmotionController** class will be called which controlles all the operations of Alchemy Extract module. Here function **getDataFromAlchemyDB()** will be executed.
   
#### Step 5:
   This **getEmotionTextData()** function gets the multiple records data from Request table and sends request to Alchemy API response function **emotion('text', $text, null)** one by one using for loop.
   
#### Step 6:
   On receiving response from API, the JSON response will be stored in Mongo DB by calling function  **insertBlueMixJSONResponseIntoMongo(json_encode($data))**.

#### Step 7:
   The response array will be inserted into Master data by function **insertEmotionMasterDataIntoMySQL($data,$id);** from Controller.
   
**_Code:_**

```  
 public function insertAllEmotionMasterDataIntoMySQL($data,$id){
		//print_r($data); exit;
		$this->response_array =$data;
		$masterId=$this->insertIntoMasterData(json_encode($data),$id);
        if($masterId>0)
		return $this->getParsedDataFromJSONResponse($masterId);
	}
	
	
	public function insertIntoMasterData($json_response,$id){
    	$sql = "INSERT INTO master_emotion_request
               (
                alm_sugar_id,
                alm_emotion_response_text,
                alm_request_date,alm_external_id
                ) VALUES (
                
                '',
                '$json_response',
                NOW(),
				'$id'
                )";
		mysqli_query($this->con,$sql);
        return $this->con->insert_id;
    }

```

#### Step 8:
   Here based on the master request id, the response will be stored in Child table by function **getParsedDataFromJSONResponse($masterReqId)**.

**_Code:_**

```
  public function getParsedDataFromJSONResponse($masterId){
            $row['master_emotion_id']=$masterId;
			$row['anger']=$this->response_array['docEmotions']['anger'];
            $row['disgust']=$this->response_array['docEmotions']['disgust']; 
		    $row['fear']=$this->response_array['docEmotions']['fear'];
			$row['joy']=$this->response_array['docEmotions']['joy'];
			$row['sadness']=$this->response_array['docEmotions']['sadness'];
			 $this->insertIntoRowMysqlTable($row);
    }

    public function insertIntoRowMysqlTable($rowData=array()){
    	$sql = "INSERT INTO child_emotion_request(
                    master_emotion_id,
					anger,
                    disgust,
                    fear,
					joy,
					sadness,
                    alc_date
                ) VALUES (
                    '".$rowData['master_emotion_id']."',
					'".$rowData['anger']."',
                    '".$rowData['disgust']."',
                    '".$rowData['fear']."',
					'".$rowData['joy']."',
					'".$rowData['sadness']."',
                    NOW()
                )";
        mysqli_query($this->con,$sql);
        return $this->con->insert_id;
    }
    
```

#### Step 9:
   On inserting the JSON response into master and child tables, the status and Request date will be updated for that record in the Request table by function **updateEmotionTextData($id)** in controller.


#### Step 10:
   To get the Master list function **getALLMasterDataFromMySQL** will be called from controller.
   
#### Step 11:
   To view the Master list function **showEmotionMasterListView()** will be called from controller to View.
   
**_Code:_**

```
  function showEmotionMasterListView($data_arr){
        $smarty = new Smarty();
        $smarty->assign('base_path',$GLOBALS['base_path']);
		$smarty->assign('cursor',$data_arr);
		
	    $smarty->display(''.$GLOBALS['root_path'].'/Views/AlchemyEmotion/allMasterList.tpl');
    }
    
``` 

#### Step 12:
   To view the Child data based on the master id, function **getEmotionChildDataFromMySQL($post_data)** will be called from controller.
   Function **getAllChildDataFromMySQL($post_data)** will get the records based on the master id using MySql query. Function **showChildDetailListView($alchemy_list_vo)** will be called in view. 
   
**_Code:_**

```
public function getEmotionChildDataFromMySQL($post_data){
        $alchemy_action = AlchemyEmotionAction::getInstance();
        $alchemy_list_vo = $alchemy_action->getAllChildDataFromMySQL($post_data);
		$alchemy_view = AlchemyEmotionView::getInstance();
    	$alchemy_view->showChildDetailListView($alchemy_list_vo);
	}
```
