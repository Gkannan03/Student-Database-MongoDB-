# Student-Database-MongoDB-
Create a database and then load the student.json dataset.
    
    import json
    from pymongo import MongoClient
    from pymongo.errors import DuplicateKeyError
    client=MongoClient("mongodb://localhost:27017/")
    mydb=client["Student"]
    mycol=mydb["Student_marks"]

    f=open('Students.json','r')
    try:
        for i in f:
            j=json.loads(i)
            mycol.insert_one(j)
    except DuplicateKeyError:
        pass

Find the student name who scored maximum scores in all (exam, quiz and homework)

    stage1={"$addFields":{"Total_marks":{"$sum":{"$sum":["$scores.score"]}}}}
    stage2={"$sort":{"Total_marks":-1}}
    stage3={"$limit":1}
    for k in mycol.aggregate([stage1,stage2,stage3]):
        print(k)
        
Find students who scored below average in the exam and pass mark is 40%?

    stage1={"$unwind":"$scores"}
    stage2={"$match":{"scores.type":"exam"}}
    stage3={"$group":{"_id":"_id", "Avg":{"$avg":"$scores.score"}}}
    for k in mycol.aggregate([stage1,stage2,stage3]):
        s=k
    stage4={"$unwind":"$scores"}
    stage5={"$match":{"scores.type":"exam"}}
    stage6={"$match":{"$and":[{"scores.type":"exam"},{"$and":[{"scores.score":{"$lt":s["Avg"]}},{"scores.score":{"$gte":40}}]}]}}
    for l in mycol.aggregate([stage4,stage6]):
        print(l)
        
 Find students who scored below pass mark and assigned them as fail, and above pass mark as pass in all the categories.
 
     for i in mycol.find():
        exam=i['scores'][0]['score']
        quiz=i['scores'][1]['score']
        homework=i['scores'][2]['score']
        if exam >= 40 and quiz >= 40 and homework >= 40:
            i.update({"Status":"Pass"})
            print(i)
        else:
            i.update({"Status": "Fail"})
            print(i)
      
 Find the total and average of the exam, quiz and homework and store them in a separate collection.
 
      stage1= {"$unwind":"$scores"}
      stage2= {"$group":{"_id":"$scores.type", "Total":{"$sum":"$scores.score"}, "Average":{"$avg":"$scores.score"}}}
      stage3= {"$out":"Total_Avg_collection"}
      for i in mycol.aggregate([stage1,stage2, stage3]):
          print(i)
          
          
 Create a new collection which consists of students who scored below average and above 40% in all the categories.
 
    stage1={"$unwind":"$scores"}
    stage2={"$group":{"_id":"$scores.type", "Average":{"$avg":"$scores.score"}}}
    average=[]
    for i in mycol.aggregate([stage1,stage2]):
        x=i['Average']
        average.append(x)
    stage={"$and":[{"$and":[{"scores.0.score":{"$gte":40}},{"scores.0.score":{"$lt":average[2]}}]},
       {"$and":[{"scores.1.score":{"$gte":40}},{"scores.1.score":{"$lt":average[0]}}]},
       {"$and":[{"scores.2.score":{"$gte":40}},{"scores.2.score":{"$lt":average[1]}}]}]}
    for i in mycol.find(stage):
        print(i)
 
 Create a new collection which consists of students who scored below the fail mark in all the categories.
 
    stage1= {'$project': {'_id': 0,'name': 1,'scores': {'$filter': {'input': "$scores",'as': "score",'cond': {'$lt': ["$$score.score", 40]}}}}}
    stage2= {'$match': {"scores.0": {'$exists': True},'$expr': {'$eq': [{'$size': "$scores"},3]}}}
    stage3= {'$out': "failed_students"}
    for i in mycol.find(stage1, stage2, stage3):
        print(i)
        
 Create a new collection which consists of students who scored above pass mark in all the categories.
 
    stage1= {'$project': {'_id': 0,'name': 1,'scores': {'$filter': {'input': "$scores",'as': "score",'cond': {'$gte': ["$$score.score", 40]}}}}}
    stage2= {'$match': {"scores.0": {'$exists': True},'$expr': {'$eq': [{'$size': "$scores"},3]}}}
    stage3= {'$out': "failed_students"}
    for i in mycol.find(stage1, stage2, stage3):
        print(i)

 
 
 
