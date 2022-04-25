- Name: Build-Source
          Actions:
            - Name: BuildSource
              ActionTypeId:
                Category: Build
                Owner: AWS
                Provider: CodeBuild
                Version: 1
              InputArtifacts:
                - Name: TemplateSource
              OutputArtifacts:
                - Name: BuildOutput
              Configuration:
                ProjectName: !Ref BuildStuff
                PrimarySource: TemplateSource
              RunOrder: 1
        - Name: Make-Change-Set
          Actions:
            - Name: MakeChangeSet
              ActionTypeId:
                Category: Deploy
                Owner: AWS
                Version: 1
                Provider: CloudFormation
              Configuration:
                ChangeSetName: !Ref ProjectName
                ActionMode: CHANGE_SET_REPLACE
                StackName: !Ref ProjectName
                Capabilities: CAPABILITY_NAMED_IAM
                TemplatePath: BuildOutput::package.yaml
                TemplateConfiguration: !Sub TemplateSource::config/${Environment}/${AWS::Region}.json
                RoleArn:
                  !FindInMap [EnvironmentsAndRoles, !Ref Environment, CfnDeploy ]
              InputArtifacts:
                - Name: BuildOutput
                - Name: TemplateSource
              RunOrder: 1
              RoleArn:
                  !FindInMap [EnvironmentsAndRoles, !Ref Environment, PipelineRole ]
        - Name: Deploy
          Actions:
            - Name: API
              ActionTypeId:
                Category: Deploy
                Owner: AWS
                Version: 1
                Provider: CloudFormation
              Configuration:
                ChangeSetName: !Ref ProjectName
                ActionMode: CHANGE_SET_EXECUTE
                StackName: !Ref ProjectName
                RoleArn:
                  !FindInMap [EnvironmentsAndRoles, !Ref Environment, CfnDeploy ]
              RoleArn:
                !FindInMap [EnvironmentsAndRoles, !Ref Environment, PipelineRole ]