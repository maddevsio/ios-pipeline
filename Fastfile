# Uncomment the line if you want fastlane to automatically update itself
# update_fastlane

default_platform(:ios)

platform :ios do
  before_all do |lane, options|
    ENV["XCODE_PROJ"] = "<XCODE_PROJ>.xcodeproj" 
    ENV["XCWORKSPACE"] = "<XCWORKSPACE>.xcworkspace"
    ENV["TEAM_ID"] = "<TEAM_ID>"
    # FORCE UPDATE XCODE - will check current version. If we don't have needed - will install it.
    xcode_install(
      version: "11.1",
      username: ENV["APPLE_ID"],
      team_id: ENV["TEAM_ID"]
    )
    # KEYCHAIN_PASSWORD - password that allows to login to this computer.
    unlock_keychain(path: "login.keychain",password: ENV["KEYCHAIN_PASSWORD"])
#    clear_derived_data
    # need to set variables before other scripts
    if options[:production] 
      setup_production()
    else
      setup_staging()
    end
    plist()
    
    # do not change order. need to fetch plist before get plist
    # this value will be stored in Actions.lane_context[SharedValues::VERSION_NUMBER]
    get_version_number(
      xcodeproj: ENV["XCODE_PROJ"],
      target: ENV["APP_NAME"]
    )
    ENV["IOS_SLACK_NAME"] = "iOS #{ENV["APP_NAME"]} v: #{Actions.lane_context[SharedValues::VERSION_NUMBER]} build (pipeline_id): #{ENV["CI_PIPELINE_ID"]}"
  end #:before_all do
  ###########################################

  lane :setup_staging do
    ENV["APP_ID"] = "<STAGING_APP_ID>"
    ENV["APP_NAME"] = "<STAGING_APP_NAME>"
    ENV["APP_ID_NAME"] = "<STAGING_APP_ID_NAME>"
    ENV["PLIST_PATH"] = "./<STAGING_PLIST_PATH>/Staging-Info.plist"
    ENV["GSP_PATH"] = "./<STAGING_GSP_PATH>/GoogleService-Info.plist"
    ENV["DELIVER_ITMSTRANSPORTER_ADDITIONAL_UPLOAD_PARAMETERS"] = "-t Aspera"
    ENV["CONFIG"] = "Release"
  end #:setup_staging do
  ###########################################

  lane :setup_production do
    ENV["APP_ID"] = "<PRODUCTION_APP_ID>"
    ENV["APP_NAME"] = "<PRODUCTION_APP_NAME>"
    ENV["APP_ID_NAME"] = "<PRODUCTION_APP_ID_NAME>"
    ENV["PLIST_PATH"] = "./<PRODUCTION_PLIST_PATH>/Info.plist"
    ENV["GSP_PATH"] = "./<PRODUCTION_GSP_PATH>/GoogleService-Info.plist"
    ENV["DELIVER_ITMSTRANSPORTER_ADDITIONAL_UPLOAD_PARAMETERS"] = "-t Aspera"
    ENV["CONFIG"] = "Release"
  end #:setup_production do
  ###########################################

  desc "Install pod dependencies"
  lane :install_pods do
    cocoapods(
      clean: true,
      podfile: "Podfile",
      try_repo_update_on_error: true
    )
  end #:install_pods do  
  ###########################################
  
  lane :plist do
    update_info_plist(
      xcodeproj:      ENV["XCODE_PROJ"],
      plist_path:     ENV["PLIST_PATH"],
      display_name:   ENV["APP_NAME"]
    )
  end #:plist do  
  ###########################################

  lane :certificates_debug do
    match(
			type: "development",
			git_url: ENV["MATCH_URL"],
			app_identifier: [<PRODUCTION_APP_ID_NAME>, <STAGING_APP_ID_NAME>],
			username: ENV["APPLE_ID"],
			readonly: true,
			verbose: true)
  end #:certificates_debug do
  ###########################################

  lane :certificates_appstore do
    match(
			type: "appstore",
			git_url: ENV["MATCH_URL"],
			app_identifier: [<PRODUCTION_APP_ID_NAME>, <STAGING_APP_ID_NAME>],
			username: ENV["APPLE_ID"],
			readonly: true,
			verbose: true)
  end #:certificates_appstore do
  ###########################################

  desc "Runs all the tests on Staging"
  lane :test do
    install_pods()
    certificates_debug()

    run_tests(
      scheme: ENV["APP_NAME"],
      output_directory: "fastlane/tests",
      suppress_xcode_output: true,
      device: "iPhone 11",
      clean: true,
      configuration: "Debug",
      reset_simulator: true,
      code_coverage: true
    )
  end #:test do
  ###########################################

  lane :wait_for_build do
    Spaceship::Tunes.login(ENV["APPLE_ID"], ENV["FASTLANE_PASSWORD"])
    
    app_version = Actions.lane_context[SharedValues::VERSION_NUMBER]
    app_build = ENV["CI_PIPELINE_ID"]
    app_id = ENV["APP_ID"]
    latest_build = FastlaneCore::BuildWatcher.wait_for_build_processing_to_be_complete(app_id: app_id, platform: "ios", app_version: app_version, build_version: app_build, poll_interval: 100, return_spaceship_testflight_build: false)

    puts app_build
    unless latest_build.app_version == app_version && latest_build.version == app_build
      UI.important("Uploaded app #{app_version} - #{app_build}, but received build #{latest_build.app_version} - #{latest_build.version}.")
    end
  end #:wait_for_build do
  ###########################################
  
  desc "Runs build and upload operations sequentially"
  lane :build_with_upload do |options|
    latest_testflight_build_number(
			version: Actions.lane_context[SharedValues::VERSION_NUMBER],
			app_identifier: ENV["APP_ID"],
			username: ENV["APPLE_ID"],
			team_id: ENV["TEAM_ID"]
    )
    
    if Actions.lane_context[SharedValues::LATEST_TESTFLIGHT_BUILD_NUMBER].to_s == ENV["CI_PIPELINE_ID"].to_s
			puts "CI PIPELINE ID equal's to latest testflight build number. No need to build app"
			next #exit(0)
    end
    
    build()
    # wait until ipa copypastes to output_directory
    sleep 100
    upload()
  end #:build_with_upload do
  ###########################################

  desc "Build app to upload for TestFlight"
  lane :build do |options|
    install_pods()
    # Bump version
    increment_build_number(build_number: ENV['CI_PIPELINE_ID'])
    # Install appstore certs
    certificates_appstore()
    build_ios_app(
      export_method: "app-store",
      workspace: ENV["XCWORKSPACE"],
      scheme: ENV["APP_NAME"],
      output_directory: "./fastlane/build",
      output_name: "#{ENV["APP_NAME"]}.ipa",
      clean: true,
      configuration: ENV["CONFIG"],
      include_bitcode: true,
      include_symbols: false,
      suppress_xcode_output: true,
      export_options: {
        provisioningProfiles: {
          "<PRODUCITON_APP_ID_NAME>" => "match AppStore <PRODUCTION_APP_ID_NAME>",
          "<STAGING_APP_ID_NAME>" => "match AppStore <STAGING_APP_ID_NAME>"
        }
      }
    )
  end #:build do
  ###########################################

  desc "Submit a new version to the TestFlight"
  lane :upload do
    latest_testflight_build_number(
          version: Actions.lane_context[SharedValues::VERSION_NUMBER],
          app_identifier: ENV["APP_ID"],
          username: ENV["APPLE_ID"],
          team_id: ENV["TEAM_ID"]
    )
    
    if Actions.lane_context[SharedValues::LATEST_TESTFLIGHT_BUILD_NUMBER].to_s == ENV["CI_PIPELINE_ID"].to_s
        puts "CI PIPELINE ID equal's to latest testflight build number. No need to upload it"
        next #exit(0)
    end
    
    changelog_from_git_commits()
    puts "Uploading new version to testflight"

    upload_to_testflight(
      username: ENV["APPLE_ID"],
      app_identifier: ENV["APP_ID_NAME"],
      skip_submission: true,
      skip_waiting_for_build_processing: true,
      ipa: "./fastlane/build/#{ENV["APP_NAME"]}.ipa"
    )
  end #:upload do
  ###########################################

  desc "Update dSym files after build processing"
  lane :refresh_dsym do
    app_version = Actions.lane_context[SharedValues::VERSION_NUMBER]
    latest_testflight_build_number(
			version: app_version,
			app_identifier: ENV["APP_ID"],
			username: ENV["APPLE_ID"],
			team_id: ENV["TEAM_ID"]
    )

    if Actions.lane_context[SharedValues::LATEST_TESTFLIGHT_BUILD_NUMBER].to_s == ENV["CI_PIPELINE_ID"].to_s
        puts "CI PIPELINE ID equal's to latest testflight build number. No need to upload it"
        next
    end
    
    wait_for_build()
    install_pods()
    
    # download dSym from appStoreConnect
    download_dsyms(
      username: ENV["APPLE_ID"],
      app_identifier: ENV["APP_ID_NAME"],
      team_id: ENV["TEAM_ID"],
      version: app_version
    )
    
    # upload dSym to Firebase Crashlytics
    upload_symbols_to_crashlytics(
      gsp_path: ENV["GSP_PATH"]
    )
    
    # remove downloaded dSym files from CI
    clean_build_artifacts()
  end #refresh_dsym do
  ###########################################

end #platform :ios do
