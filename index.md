<h1 align="center">Plus Content Backends</h1>

<h2>I. About Config</h2>
<div style="margin: 0 0 0 20px;">
  <h3 style="color:yellow"> * About Local Environments</h3>

  <div style="margin: 0 0 0 15px">
    <span style="color:greenyellow">- LocalStack via S3 local</span>
    <pre>
      AWS_ACCESS_KEY_ID=AWSACCESSKEYIDEXAMPLE
      AWS_SECRET_ACCESS_KEY=AWSSECRETACCESSKEYEXAMPLE
      AWS_DEFAULT_REGION=ap-northeast-1
      AWS_BUCKET=pluscontents-local-data
      AWS_ENDPOINT=http://localstack.contents.plus-msg.auone.local:4566
    </pre>
    <span style="color:greenyellow"> - SFTP</span>
    <pre>
      ip publish: 3.112.214.42
      username: sftp_user
      password: SftpUser@123S
    </pre>
  </div>

</div>

<h2>II. How to fake data ?</h2>

<div style="margin: 0 0 0 20px;">
  <span style="color:greenyellow">- Fake Members Data</span>
  <pre>
    docker-compose exec stub php artisan quiz:fake:member {numberRecord}</pre>

  <span style="color:greenyellow">- Fake QuizRecord Data</span>
  <pre>docker-compose exec stub php artisan quiz:fake:quiz_record --option
  
    * with Optional
  
    {--B|begin : Fake record for only begin got from GMO} : -B or --begin
    {--S|sent : Fake record for only has sent to Tame} : -S or --sent
    {--E|exported : Fake record exported}: -E or --exported
    {--C|compare : Fake record for only has compare from Tame} : -C or --compare
    {--A|all : Fake record for all option} : -A or --all
  </pre>
  <span style="color:greenyellow">- Fake Result file and upload to Sftp</span>
  <pre>docker-compose exec stub php artisan quiz:fake:response_file {numberRecord} {targetDate}</pre>

  <ul>
    <li>*numberRecord: is record will render to excel file.</li>
    <ul>
      <li>example: 0.004 <=> 0.004*100000 = 400 records render to 1 excel file</li>
      <li>example: 2 <=> 2*100000 = 200.000 records render to 2 excel file</li>
    </ul>
    <li>*targetDate: is date condition for query sql to get from database</li>
    <li>Note*: If in Database is not enough record quantity. In bash will continue fake data when enough numberRecord param.but it not insert to DB</li>
  </ul>

</div>

<h2>III. About Run Bash ?</h2>

<div style="margin: 0 0 0 20px;">
  
  <span style="color:greenyellow">- `Health check` batch for trial run </span>
  <pre>BATCH_CMD="quiz:health-check" docker-compose up batch</pre>

  <span style="color:greenyellow">- About 01. Add point command </span>
  <pre>BATCH_CMD="quiz:add-points" docker-compose up batch</pre>

  <span style="color:greenyellow">- About 02. Point grant compare request </span>
  <pre>BATCH_CMD="quiz:request-compare {target_date}" docker-compose up batch</pre>

  <span style="color:greenyellow">- About 03.Point grant result compare batch. </span>
  <pre>BATCH_CMD="quiz:result-compare {target_date}" docker-compose up batch</pre>
  <span style="color:greenyellow">- About 04.Point grant accepting reception.</span>
  <pre>BATCH_CMD="quiz:result-accepting-compare {target_date}" docker-compose up batch</pre>
</div>

<h2>IV. How to Test Command?</h2>

<div style="margin: 0 0 0 20px;">
  <span style="color:greenyellow">- Test command 01. Add-point </span>
  <ul style="list-style:number">
    <li>Fake Member Record <pre>docker-compose exec stub php artisan quiz:fake:member 1000</pre>
    </li>
    <li>Fake QuizRecord Record Begin
      <pre>docker-compose exec stub php artisan quiz:fake:quiz_record -B</pre>
    </li>
    <li>Run sent request to tameru (use api tameru make by stub)
      <pre>BATCH_CMD="quiz:add-points" docker-compose up batch</pre>
    </li>
  </ul>
  <span style="color:greenyellow">- Test command 02. Request Compare </span>
  <ul style="list-style:number">
    <li>Fake Member Record <pre>docker-compose exec stub php artisan quiz:fake:member 1000</pre>
    </li>
    <li>Fake QuizRecord Record Sent To Tameru
      <pre>docker-compose exec stub php artisan quiz:fake:quiz_record -S</pre>
    </li>
    <li>Run sent request to tameru (use api tameru make by stub)
      <pre>BATCH_CMD="quiz:request-compare {target_date}" docker-compose up batch</pre>
      <pre>BATCH_CMD="quiz:request-compare" docker-compose up batch</pre>
      <i>using target_date: is date created_at in step2 (fake QuizRecord) </i>
    </li>
    <li>Check File & type File
      <pre>ssh sftp_user@sftp_ip_server
ls -la /request/</pre>
    </li>
  </ul>


  <span style="color:greenyellow">- Test command 03.Point grant result compare batch. </span>
  <ul style="list-style:number">
    <li>Fake Member Record <pre>docker-compose exec stub php artisan quiz:fake:member 2000</pre>
    </li>
    <li>Fake QuizRecord Record Sent to Tameru
      <pre>docker-compose exec stub php artisan quiz:fake:quiz_record -E</pre>
    </li>
    <li>Fake Result File and upload to Sftp server use Dev env.
      <pre>docker-compose exec stub php artisan quiz:fake:response_file 0.005 {target_date}</pre>
      <i>using target_date: is date created_at in step2 (fake QuizRecord) </i>
    </li>
    <li>Run 03.Point grant result compare batch.
      <pre>BATCH_CMD="quiz:result-compare {target_date}" docker-compose up batch</pre>
      <i>using target_date: is date created_at in step2 (fake QuizRecord) </i>
    </li>
  </ul>

  <span style="color:greenyellow">- Test command 04.point grant accepting reception. </span>
  <ul style="list-style:number">
    <li>Fake Member Record <pre>docker-compose exec stub php artisan quiz:fake:member 2000</pre>
    </li>
    <li>Fake QuizRecord Record Sent to Tameru
      <pre>docker-compose exec stub php artisan quiz:fake:quiz_record -E</pre>
    </li>
    <li>Fake Result File and upload to Sftp server use Dev env.
      <pre>docker-compose exec stub php artisan quiz:fake:response_file 500 {target_date}</pre>
      <i>using target_date: is date created_at in step2 (fake QuizRecord) </i>
    </li>
    <li>Run 04.Point grant accepting reception.
      <pre>BATCH_CMD="quiz:result-accepting-compare {target_date}" docker-compose up batch</pre>
      <i>using target_date: is date created_at in step2 (fake QuizRecord) </i>
    </li>
  </ul>

</div>
